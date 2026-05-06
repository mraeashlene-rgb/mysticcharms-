# Complete Rating System Implementation Guide

## Overview
A full-featured Firebase-based rating system with image uploads, real-time database storage, and homepage display.

## System Components

### 1. **Firebase Storage** (Images)
- **Path**: `order-feedback/{orderId}/{imageName}`
- **File Naming**: `{timestamp}_{randomId}_{index}.{extension}`
- **Metadata**: Stores uploadedAt and orderId
- **Max Files**: 5 images per rating
- **File Types**: Images only (validated)

### 2. **Firebase Realtime Database** (Order Ratings)
Each order stores rating data with:
```
{
  rating: 1-5 (number),
  feedback: "comment text",
  feedbackImages: ["url1", "url2", ...],
  customerName: "M*****y" (masked),
  date: "ISO-8601 timestamp",
  orderId: "order-id",
  userId: "user-id",
  hasRated: true,
  ratedAt: timestamp,
  status: "delivered" or "completed"
}
```

## Workflow

### Orders Page (`orders.html`)

#### 1. **Load Orders**
```javascript
loadOrders(currentUser)
  ├─ Fetches user's orders from FirebaseDB
  ├─ Filters: status === "delivered" || "completed"
  ├─ Stores in ordersById object for quick access
  └─ Displays active & completed orders
```

#### 2. **Show Rating Options**
- **Not Rated**: Shows "Rate Your Order" button
- **Rated**: Shows rating summary with stars, feedback, and images
- **Rated**: Shows "Edit Rating" button

#### 3. **Open Rating Modal**
```javascript
openRatingModal(isEdit, orderId)
  ├─ If editing: Load existing rating, feedback, images
  ├─ If new: Clear all fields
  ├─ Show modal with:
  │   ├─ Star selector (1-5)
  │   ├─ Feedback textarea
  │   ├─ Image upload (max 5)
  │   └─ Submit button
  └─ Update modal title based on mode
```

#### 4. **Handle Image Selection**
```javascript
handleImageSelection(event)
  ├─ Validate all files are images
  ├─ Warn if >5 files selected
  ├─ Store in selectedFiles array
  └─ Display preview with remove buttons
```

#### 5. **Submit Rating**
```javascript
handleSubmitRating()
  ├─ Validate: rating >= 1, feedback not empty
  ├─ Upload new images to Firebase Storage
  │   └─ Get download URLs for each image
  ├─ Combine existing + new images (max 5)
  ├─ Mask customer name (e.g., R*****E)
  ├─ Save to Firebase Realtime DB:
  │   ├─ orders/{orderId} (update order)
  │   └─ ratings/{orderId} (duplicate for easy retrieval)
  ├─ Show success message
  ├─ Close modal
  ├─ Refresh orders display
  └─ Button state: "Rate Your Order" → "Edit Rating"
```

### Homepage (`index.html`)

#### 1. **Fetch Ratings**
```javascript
onValue(ref(db, "orders"))
  ├─ Listen for real-time updates
  ├─ Filter: (hasRated || rating > 0) AND (status === "delivered" || "completed")
  ├─ Sort by ratedAt (newest first)
  └─ Render review cards
```

#### 2. **Display Reviews**
Each review card shows:
- ⭐ Star rating (1-5 stars)
- 🖼️ Image carousel (if images exist)
  - Left/Right navigation buttons
  - Click handlers for image navigation
- 👤 Masked customer name
- 💬 Feedback comment

#### 3. **Image Navigation**
```javascript
reviewsGrid.addEventListener("click")
  ├─ Detect carousel navigation buttons
  ├─ Calculate next/prev slide index
  ├─ Update CSS transform to show slide
  └─ Reset index to 0 when reaching end
```

## Key Functions

### Image Upload
```javascript
uploadSelectedImages(orderId)
  → Array of download URLs from Firebase Storage
  → Error handling with partial upload support
```

### Name Masking
```javascript
maskCustomerName(name)
  → "John Doe" → "J***e"
  → "Alice" → "A***e"
  → Preserves first and last character
```

### Security Functions
```javascript
escapeHtml(text)          → Prevents XSS attacks
escapeAttribute(url)      → Safe HTML attributes
```

## Data Flow Diagram

```
User submits rating with images
    ↓
Validate input
    ↓
Upload images to Firebase Storage
    ↓
Get download URLs
    ↓
Save rating to Firebase RealtimeDB
    ├── /orders/{orderId}
    └── /ratings/{orderId}
    ↓
Show success message
    ↓
Refresh orders page
    ↓
Homepage real-time listener triggers
    ↓
Render review cards with images
```

## File Structure

### orders.html
- Rating modal UI
- Image upload/preview
- Submit handling
- Modal state management
- Orders list display with ratings

### index.html
- Reviews grid display
- Image carousel component
- Real-time ratings listener
- Review card rendering

## Features

✅ **Image Uploads**
- Up to 5 images per rating
- File type validation
- Preview with remove buttons
- Progress feedback ("Uploading...")

✅ **Rating Management**
- Create new ratings
- Edit existing ratings
- Add/remove images when editing
- Mask customer names for privacy

✅ **Real-time Updates**
- Firebase Realtime Database listener
- Instant display of new ratings on homepage
- Live refresh on orders page

✅ **Error Handling**
- Non-image file validation
- Upload failure recovery (partial uploads)
- Network error handling
- User-friendly error messages

✅ **Security**
- HTML escaping to prevent XSS
- Temporary object URLs for previews
- Secure image metadata storage

## Testing Checklist

- [ ] Submit rating with stars and comment
- [ ] Upload multiple images
- [ ] Edit rating - add new images
- [ ] Edit rating - remove existing images
- [ ] View rating on homepage immediately
- [ ] Image carousel navigation works
- [ ] Customer name is masked
- [ ] Error handling with invalid files
- [ ] Max 5 images enforced
- [ ] Success message appears

## Firebase Rules Required

```json
{
  "rules": {
    "orders": {
      "$orderId": {
        ".read": "root.child('users').child(auth.uid).exists()",
        ".write": "root.child('users').child(auth.uid).exists()"
      }
    },
    "ratings": {
      "$ratingId": {
        ".read": true,
        ".write": "root.child('users').child(auth.uid).exists()"
      }
    },
    "order-feedback": {
      "$orderId": {
        "$fileName": {
          ".write": "auth.uid != null",
          ".read": true
        }
      }
    }
  }
}
```

## Troubleshooting

**Issue**: Images not uploading
- Check Firebase Storage rules
- Verify bucket name in config
- Check browser console for CORS errors

**Issue**: Rating not showing on homepage
- Verify hasRated flag is being set
- Check order status is "delivered" or "completed"
- Clear browser cache and refresh

**Issue**: Modal not opening
- Verify order ID is present
- Check browser console for JS errors
- Ensure Firebase is initialized

**Issue**: Images not displaying
- Check image URLs are accessible
- Verify Firebase Storage is public readable
- Check CORS configuration

---

Last Updated: April 20, 2026
