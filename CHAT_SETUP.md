# Live Chat Feature

## Overview
A real-time chat system has been integrated into your multiplayer game, allowing players to communicate while in the game lobby before starting a match.

## Features
- **Real-time messaging** using Cloud Firestore
- **Auto-scrolling** chat display
- **Message timestamps** for each message
- **Player identification** with unique player names
- **Responsive UI** that adapts to lobby screen layout
- **Message persistence** per game room

## Files Created

### 1. **lib/models/chat_message.dart**
Data model for chat messages with:
- `senderId` - User's Firebase UID
- `senderName` - Display name of the player
- `message` - The message content
- `timestamp` - When the message was sent
- Serialization methods for Firestore

### 2. **lib/services/chat_service.dart**
Core chat functionality:
- `sendMessage()` - Send a message to a room's chat
- `getMessagesStream()` - Stream of messages ordered chronologically
- `deleteRoomChat()` - Clean up messages when room closes
- Uses Firestore subcollection: `rooms/{roomId}/chat`

### 3. **lib/widgets/chat_widget.dart**
UI component displaying:
- Message list with auto-scroll to latest
- Message bubbles with sender name and timestamp
- Input field with send button
- Left-aligned bubbles for other players
- Right-aligned bubbles (blue) for your messages
- Empty state message

## Firebase Firestore Structure
```
rooms/
  {roomId}/
    chat/
      {messageId}/
        senderId: string
        senderName: string
        message: string
        timestamp: timestamp
```

## Usage in Lobby

The chat widget is automatically integrated into the lobby screen when a room is created/joined:
- **Left side (40%)**: Room information and player list
- **Right side (60%)**: Live chat between players

Players can type messages immediately after creating or joining a room.

## How to Use

### For Players
1. Create or join a game room
2. Chat appears on the right side of the screen
3. Type messages in the input field at the bottom
4. Press enter or tap the send button to send
5. Messages appear in real-time for all players

### Integration Points
The chat is integrated into:
- `lib/lobby_screen.dart` - Main integration point
  - Initializes `ChatService` in `initState()`
  - Displays `ChatWidget` when a room is active
  - Passes `roomId`, `playerName`, and `chatService`

## Data Storage (Firestore)
- Messages are stored in: `rooms/{roomId}/chat/{docId}`
- Structure: sent by user ID, with timestamp for ordering
- No manual cleanup needed - messages persist for room history
- Optional: Call `chatService.deleteRoomChat(roomId)` to clear messages

## Security Rules (Firebase Console)
Recommended Firestore rules:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{roomId}/chat/{document=**} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && 
                       request.resource.data.senderId == request.auth.uid;
      allow delete: if request.auth != null && 
                       resource.data.senderId == request.auth.uid;
    }
  }
}
```

## Testing
1. Run the app: `flutter run`
2. Create a room (or join one)
3. Chat widget appears on the right
4. Type and send messages
5. Messages sync in real-time across all connected clients

## Future Enhancements
- Message search/filtering
- Emoji support
- Read receipts
- Typing indicators
- Message reactions
- Private DM functionality
- Chat history limit (e.g., last 100 messages)
- Chat moderation features
