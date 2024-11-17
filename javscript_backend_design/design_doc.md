# API Design Document

## Overview

This document outlines the API design for a Content or posts feed system for social media app like Insta/FB.

## Goals

The goal of this API is to provide a simple and efficient way for users to see and interact with posts created by their friends.

## Requirements

1. A user can see only those posts which were created by their friends.
2. A user can see posts of people who are not their friends, only if a friend has posted a comment on it.

## Features

## API Setup - Authentication and Authorization, Database Schema, Middleware, Error Handling

1. #### Database Schema

- We will be considering the Database schema given as a answer to question 1 for this api design.

2. #### Authentication and Authorization

- User authentication using JWT tokens, with refresh tokens for long-lived sessions.
- Data Validation and error handling.
- Hashing and salting user passwords for security.

3. #### Middleware and utilities

- Auth middleware for verifying JWT tokens.
- Multer middleware for handling file uploads.
- Error and Async handling utility for handling errors.
- Cloudinary utility for handling image uploads.
- API Response utility for consistent API responses.
- API Error utility for handling API errors.

## API Endpoints
```
```
1. ### Auth Route
   ## `api/auth`
    #### 1.Register User
    - `api/auth/register`
    - Method: POST
    - Description: Register a new user.
    - Request Body:
    ```json
    {
        "username": "username",
        "email": "email",
        "password": "password"
    }
    ```
    - Some JavaScript code snippets
    - `app.js`
    ```js
      app.use('/api/v1/auth', authRouter);
      ```
    - `auth.routes.js`
    ```js
    const router = require('express').Router();
    router.route('/register').post(authController.register);
    ```
    - `auth.controller.js`
    ```js
        const registerUser = asyncHandler( async (req, res) => {
    // get user details from frontend
    // validation - not empty
    // check if user already exists: username, email
    // check for images, check for avatar
    // upload them to cloudinary, avatar
    // create user object - create entry in db
    // remove password and refresh token field from response
    // check for user creation
    // return res


    const {fullName, email, username, password } = req.body
    //console.log("email: ", email);

    if (
        [fullName, email, username, password].some((field) => field?.trim() === "")
    ) {
        throw new ApiError(400, "All fields are required")
    }

    const existedUser = await User.findOne({
        $or: [{ username }, { email }]
    })

    if (existedUser) {
        throw new ApiError(409, "User with email or username already exists")
    }
    //console.log(req.files);

    const avatarLocalPath = req.files?.avatar[0]?.path;
    //const coverImageLocalPath = req.files?.coverImage[0]?.path;

    let coverImageLocalPath;
    if (req.files && Array.isArray(req.files.coverImage) && req.files.coverImage.length > 0) {
        coverImageLocalPath = req.files.coverImage[0].path
    }
    

    if (!avatarLocalPath) {
        throw new ApiError(400, "Avatar file is required")
    }

    const avatar = await uploadOnCloudinary(avatarLocalPath)
    const coverImage = await uploadOnCloudinary(coverImageLocalPath)

    if (!avatar) {
        throw new ApiError(400, "Avatar file is required")
    }
   

    const user = await User.create({
        fullName,
        avatar: avatar.url,
        coverImage: coverImage?.url || "",
        email, 
        password,
        username: username.toLowerCase()
    })

    const createdUser = await User.findById(user._id).select(
        "-password -refreshToken"
    )

    if (!createdUser) {
        throw new ApiError(500, "Something went wrong while registering the user")
    }

    return res.status(201).json(
        new ApiResponse(200, createdUser, "User registered Successfully")
    )

    } )
    ```
    - Response:
    ```json
    {
        "statusCode": 200,
        "message": "User registered successfully",
        "data": {
            "userId": "userId",
            "username": "username",
            "email": "email",
        }
        "success": true
    }
    ```
    #### 2.Login User
    - `api/auth/login`
    - Method: POST
    - Description: Login a user.
    - Request Body:
    ```json
    {
        "email": "email",//or username
        "password": "password"
    }
    ```
    - Response: user details and tokens as JSON and HttpOnly cookies
    ```json
    {
        "statusCode": 200,
        "message": "User logged in successfully",
        "data": {
            "user": {
                "userId": "userId",
                "username": "username",
                "email": "email",
            }
            "refreshToken": "token",
            "accessToken": "token",
        }
        "success": true
    }
    ```
    #### 3.Logout User
    - `api/auth/logout`
    - Method: POST
    - Description: Logout a user.
    - Request Body: None
    - Response: Success message
    ```json
    {
        "statusCode": 200,
        "message": "User logged out successfully",
        "success": true
    }
    ```

    #### 4.Get Current User
    - `api/auth/me`
    - Method: GET
    - Description: Get current user details.
    - Request Body: None
    - Response: user details as JSON
    ```json
    {
        "statusCode": 200,
        "message": "User details fetched successfully",
        "data": {
            "userId": "userId",
            "username": "username",
            "email": "email",
        }
        "success": true
    }
    ```
    #### 5.Refresh Token
    - `api/auth/refresh`
    - Method: POST
    - Description: Refresh access token using refresh token.
    - Request Body: None
    - Response: New access token and HttpOnly cookie
    ```json
    {
        "statusCode": 200,
        "message": "Access token refreshed successfully",
        "data": {
            "accessToken": "token",
        }
        "success": true
    }
    ```

2. ### Post Route
   ## `api/v1/posts`
    #### 1.Create Post
    - `api/v1/posts`
    - Method: POST
    - Description: Create a new post.
    - Request Body:
    ```json
    {
        "content": "content",
        "image": "image",
    }
    ```
    - Javascript code snippets
    - `app.js`
    ```js
    app.use('/api/v1/posts', postRouter);
    ```
    - `post.routes.js`
    ```js
    const router = require('express').Router();
    router.route('/').post(postController.createPost);
    ```
    - `post.controller.js`
    ```js
    const createPost = asyncHandler( async (req, res) => {
    // get post details from frontend
    // validation - not empty
    // upload image to cloudinary
    // create post object - create entry in db
    // return res
    const {userId, content} = req.body

    const post = await Post.create({
        user: userId,
        content,
    })

    
    return res.status(201).ApiResponse(200, post, "Post created Successfully")
    } )
    ```
    - Response: post details as JSON
    - Example:
    ```json
    {
        "statusCode": 200,
        "message": "Post created successfully",
        "data": {
            "postId": "postId",
            "content": "text",
            "userId": "userId",
            "createdAt": "createdAt",
        }
        "success": true
    }
    ```
    #### 2.Get Posts for User feed
    - `api/v1/posts/feed/?userId=userId&page=1&limit=10`
    - Method: GET
    - Query Parameters: userId, page, limit
    - Description: Get posts for a user's feed.
    - Request Body: None
    - Response: posts as JSON
    - Example:
    ```json
    ```
    #### 3.Get a single Post
    - `api/v1/posts/:postId`
    - Method: GET
    - Description: Get a single post.
    - Request Body: None
    - Response: post details as JSON
    - Example:
    ```json
    ```

    #### 4.Update a Post
    - `api/v1/posts/:postId`
    - Method: PATCH
    - Description: Update a post.
    - Request Body:
    ```json
    ```
    - Response: post details as JSON
    - Example:
    ```json
    ```

    #### 5.Delete a Post
    - `api/v1/posts/:postId`
    - Method: DELETE
    - Description: Delete a post.
    - Request Body: None
    - Response: Success message
    - Example:
    ```json
    ```

3. ### User Route
   ## `api/v1/users`
    #### 1.Get a single User
    - `api/v1/users/:userId`
    - Method: GET
    - Description: Get a single user.
    - Request Body: None
    - Response: user details as JSON
    - Example:
    ```json
    ```

    #### 2.Update a User
    - `api/v1/users/:userId`
    - Method: PATCH
    - Description: Update a user.
    - Request Body:
    ```json
    ```
    - Response: user details as JSON
    - Example:
    ```json
    ```

    #### 3.Delete a User
    - `api/v1/users/:userId`
    - Method: DELETE
    - Description: Delete a user.
    - Request Body: None
    - Response: Success message
    - Example:
    ```json
    ```
4. ### Friend Route
   ## `api/v1/friends`
    #### 1.Send Friend Request
    - `api/v1/friends/request`
    - Method: POST
    - Description: Send a friend request.
    - Request Body:
    ```json
    ```
    - Response: friend request details as JSON
    - Example:
    ```json
    ```

    #### 2.Accept Friend Request
    - `api/v1/friends/accept`
    - Method: POST
    - Description: Accept a friend request.
    - Request Body:
    ```json
    ```
    - Response: friend request details as JSON
    - Example:
    ```json
    ```

    #### 3.Reject Friend Request
    - `api/v1/friends/reject`
    - Method: POST
    - Description: Reject a friend request.
    - Request Body:
    ```json
    ```
    - Response: friend request details as JSON
    - Example:
    ```json
    ```

    #### 4.Get Friend Requests
    - `api/v1/friends/requests`
    - Method: GET
    - Description: Get friend requests.
    - Request Body: None
    - Response: friend request details as JSON
    - Example:
    ```json
    ```

    #### 5.Get Friends
    - `api/v1/friends`
    - Method: GET
    - Description: Get friends.
    - Request Body: None
    - Response: friend details as JSON
    - Example:
    ```json
    ```

5. ### Comment Route
   ## `api/v1/comments`
    #### 1.Create Comment
    - `api/v1/comments`
    - Method: POST
    - Description: Create a new comment.
    - Request Body:
    ```json
    ```
    - Response: comment details as JSON
    - Example:
    ```json
    ```

    #### 2.Get Comments for a Post
    - `api/v1/comments/:postId`
    - Method: GET
    - Description: Get comments for a post.
    - Request Body: None
    - Response: comments as JSON
    - Example:
    ```json
    ```
    #### 3.Delete a Comment
    - `api/v1/comments/:commentId`
    - Method: DELETE
    - Description: Delete a comment.
    - Request Body: None
    - Response: Success message
    - Example:
    ```json
    ```

6. ### Like Route
   ## `api/v1/likes`
    #### 1.Like a Post
    - `api/v1/likes`
    - Method: POST
    - Description: Like a post.
    - Request Body:
    ```json
    ```
    - Response: like details as JSON
    - Example:
    ```json
    ```

    #### 2.Get Likes for a Post
    - `api/v1/likes/:postId`
    - Method: GET
    - Description: Get likes for a post.
    - Request Body: None
    - Response: likes as JSON
    - Example:
    ```json
    ```

    #### 3.Unlike a Post
    - `api/v1/likes/:postId`
    - Method: DELETE    
    - Description: Unlike a post.
    - Request Body: None
    - Response: Success message
    - Example:
    ```json
    ```
