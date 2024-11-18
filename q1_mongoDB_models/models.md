# MongoDB Models
by Anubhav Sharma
**email** - `anubhavsharma527@gmail.com`
- This document outlines the MongoDB models social media app like Insta/FB use for.
    1) Register user details 
    2) Send/Receive and Accept/Reject friend request from users
    3) Create text-only posts and comments on their friend's posts.
- This is also the model used for the javascript api design.

## User Model
```js
const userSchema = new mongoose.Schema({
    
    username: {
        type: String,
        required: true,
        unique: true,
        index: true
    },
    firstName: {
        type: String,
    },
    lastName: {
        type: String,
    },
    profilePic: {
        type: String,
    }
    coverPic: {
        type: String,
    },
    bio: {
        type: String,
    },
    email: {
        type: String,
        required: true,
        unique: true,
        index: true
    },
    password: {
        type: String,
        required: true
    }
    
},
{
    timestamps: true
})

```

## Post Model
```js
const postSchema = new mongoose.Schema({
    
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true,
        index: true
    },
    text: {
        type: String,
        required: true
    },
    likeCount: {
        type: Number,
        default: 0
    },
    commentCount: {
        type: Number,
        default: 0
    }
},
{
    timestamps: true
})

```
## Comment Model
```js
const commentSchema = new mongoose.Schema({
    
    post: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Post',
        required: true,
        index: true
    },
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true,
        index: true
    },
    content: {
        type: String,
        required: true
    },
    likeCount: {
        type: Number,
        default: 0
    }
},
{
    timestamps: true
})

```

## Like Model
```js
const likeSchema = new mongoose.Schema({
    
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true,
    },
    post: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Post',
        required: true,
        index: true
    },
},
{
    timestamps: true
})

```

## Friends Model
```js
const friendsSchema = new mongoose.Schema({
    
    user1: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true,
        index: true
    },
    user2: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true,
        index: true
    },
    status: {
        type: String,
        enum: ['pending', 'accepted','rejected','blocked'],
        required: true
    }
},
{
    timestamps: true
})

```