---
name: mongodb-patterns
description: MongoDB patterns for document design, queries, and aggregations. Use when working with MongoDB databases.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# MongoDB Patterns

## Document Design

```ts
interface User {
  _id: ObjectId;
  email: string;
  name: string;
  profile: {
    bio?: string;
    avatar?: string;
    settings: {
      notifications: boolean;
      theme: 'light' | 'dark';
    };
  };
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
}
```

## Mongoose Schema

```ts
import { Schema, model } from 'mongoose';

const userSchema = new Schema<User>({
  email: { 
    type: String, 
    required: true, 
    unique: true,
    lowercase: true,
    trim: true
  },
  name: { type: String, required: true },
  profile: {
    bio: String,
    avatar: String,
    settings: {
      notifications: { type: Boolean, default: true },
      theme: { type: String, enum: ['light', 'dark'], default: 'light' }
    }
  },
  tags: [{ type: String }],
}, { 
  timestamps: true,
  toJSON: { virtuals: true }
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });
userSchema.index({ tags: 1 });
userSchema.index({ 'profile.settings.theme': 1 });

export const User = model<User>('User', userSchema);
```

## Query Patterns

```ts
// Find with projection
const users = await User.find({ tags: 'premium' })
  .select('email name profile.avatar')
  .sort({ createdAt: -1 })
  .limit(20)
  .lean();

// Find one with population
const user = await User.findById(userId)
  .populate('orders', 'total status')
  .lean();

// Update with options
const updated = await User.findOneAndUpdate(
  { _id: userId },
  { 
    $set: { name: 'New Name' },
    $push: { tags: 'verified' },
    $inc: { loginCount: 1 }
  },
  { new: true, runValidators: true }
);

// Upsert
await User.updateOne(
  { email: 'user@example.com' },
  { $setOnInsert: { createdAt: new Date() }, $set: { name: 'John' } },
  { upsert: true }
);
```

## Aggregation Pipeline

```ts
// User stats by month
const stats = await User.aggregate([
  { $match: { createdAt: { $gte: startDate } } },
  { 
    $group: { 
      _id: { $dateToString: { format: '%Y-%m', date: '$createdAt' } },
      count: { $sum: 1 },
      premiumCount: { 
        $sum: { $cond: [{ $in: ['premium', '$tags'] }, 1, 0] }
      }
    }
  },
  { $sort: { _id: 1 } }
]);

// Lookup (join)
const usersWithOrders = await User.aggregate([
  { $match: { _id: userId } },
  {
    $lookup: {
      from: 'orders',
      localField: '_id',
      foreignField: 'userId',
      as: 'orders'
    }
  },
  { $unwind: { path: '$orders', preserveNullAndEmptyArrays: true } }
]);
```

## Spring Data MongoDB

```java
@Document(collection = "users")
@Data
public class User {
    @Id
    private String id;
    
    @Indexed(unique = true)
    private String email;
    
    private String name;
    private Profile profile;
    private List<String> tags;
    
    @CreatedDate
    private Instant createdAt;
    
    @LastModifiedDate
    private Instant updatedAt;
}

public interface UserRepository extends MongoRepository<User, String> {
    Optional<User> findByEmail(String email);
    List<User> findByTagsContaining(String tag);
    
    @Query("{ 'profile.settings.theme': ?0 }")
    List<User> findByTheme(String theme);
}
```

## Best Practices

- Embed related data for read-heavy patterns
- Reference for many-to-many or large sub-documents
- Create compound indexes for common queries
- Use `lean()` for read-only queries (better performance)
- Avoid unbounded arrays (use bucketing or reference)
- Use aggregation for complex queries
- Set appropriate read/write concerns for consistency needs
