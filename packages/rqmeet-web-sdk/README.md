# RQMeet Web SDK

A TypeScript SDK for integrating RQMeet video meetings into web applications. Build powerful video conferencing features with a simple, intuitive API.

[![npm version](https://img.shields.io/npm/v/@rqmeet/web-sdk.svg)](https://www.npmjs.com/package/@rqmeet/web-sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

🚀 **[Live Demo](https://developer-demo.rqmeet.com)** – Try the SDK in action!

## Features

- 🎥 **Video Meetings** - Full video conferencing with audio, video, and screen sharing
- 🎨 **Drop-in Components** - Pre-built React components for rapid integration
- 👥 **Participant Management** - Role-based permissions (Host, Co-Host, Participant)
- 🚪 **Waiting Room** - Secure waiting room with host approval workflow
- 💬 **Chat** - Real-time messaging during meetings
- 🔴 **Recording** - Start and stop meeting recordings
- 🖼️ **Virtual Backgrounds** - Blur or custom image backgrounds
- 📱 **Responsive** - Optimized mobile controls and layouts
- 🌍 **Localization** - Built-in support for English (LTR) and Arabic (RTL)
- 🔒 **Secure** - Token-based authentication with invite links

## What's New in v1.2.2

- 📱 **Mobile Optimized Controls**: Better layout for mobile devices with a bottom sheet menu for secondary actions.
- 🌍 **Localization Support**: Added built-in support for Arabic (RTL) including UI mirroring.
- 👤 **Default Name**: New `defaultName` prop to pre-fill participant name in PreJoin screen.
- 🧪 **Improved Responsiveness**: Better adaptation to small screens and window resizing.
- 🛠️ **Bug Fixes**: improved stability and error handling.

## Getting Started

1. **Register** at [developer.rqmeet.com](https://developer.rqmeet.com) to create your account
2. **Get your API key** from the dashboard
3. **Install the SDK** and start building!

## Installation

```bash
npm install @rqmeet/web-sdk
```

## Quick Start

### Option 1: Drop-in Component (Recommended)

The easiest way to add video meetings to your React app:

```tsx
import { RQMeetClient, MeetingView } from '@rqmeet/web-sdk';

// The SDK automatically detects the environment:
// - http://localhost:3000 for local development
// - https://developer.rqmeet.com for production
const client = new RQMeetClient({
  apiKey: 'your_api_key'
});

function App() {
  return (
    <MeetingView
      client={client}
      meetingId="meeting_abc123"
      inviteToken="invite_token_from_api"
      branding={{
        logo: '/your-logo.png',
        primaryColor: '#6366f1',
        title: 'My Meeting'
      }}
      defaultName="User Name"            // Optional: Pre-fill display name
      language="en"                      // Optional: 'en' or 'ar'
      direction="ltr"                    // Optional: 'ltr' or 'rtl'
      onLeave={() => console.log('Left meeting')}
      onMeetingEnded={() => console.log('Meeting ended')}
      onError={(error) => console.error(error)}
    />
  );
}
```

### Option 2: Individual Components

For more control, use the `PreJoin` and `MeetingRoom` components separately:

```tsx
import { 
  RQMeetClient, 
  PreJoin, 
  MeetingRoom, 
  RQMeetRoom,
  ParticipantRole,
  SocketPermissions 
} from '@rqmeet/web-sdk';

function CustomMeeting() {
  const [room, setRoom] = useState<RQMeetRoom | null>(null);
  const [joinData, setJoinData] = useState<{
    role: ParticipantRole;
    permissions: SocketPermissions;
  } | null>(null);

  const handleJoin = async (data) => {
    const newRoom = new RQMeetRoom(
      meetingId,
      data.serverUrl,
      data.token,
      {
        audioEnabled: data.audioEnabled,
        videoEnabled: data.videoEnabled,
        virtualBackground: data.virtualBackground
      }
    );
    await newRoom.connect();
    setRoom(newRoom);
    setJoinData({ role: data.role, permissions: data.permissions });
  };

  if (!room) {
    return (
      <PreJoin
        client={client}
        meetingId="meeting_abc123"
        inviteToken="invite_token"
        onJoin={handleJoin}
      />
    );
  }

  return (
    <MeetingRoom
      room={room}
      client={client}
      meetingId="meeting_abc123"
      role={joinData.role}
      permissions={joinData.permissions}
      onLeave={() => setRoom(null)}
    />
  );
}
```

### Option 3: Programmatic Control

For full control without React components:

```typescript
import { RQMeetClient, RQMeetRoom, RoomEvent } from '@rqmeet/web-sdk';

const client = new RQMeetClient({
  apiKey: 'your_api_key'
});

// Create a meeting
const meeting = await client.createMeeting({
  name: 'Team Standup',
  maxParticipants: 10,
  enableRecording: true,
  enableChat: true
});

// Join the meeting
const room = await client.joinMeeting({
  meetingId: meeting.id,
  participantName: 'John Doe',
  participantId: 'user_123',
  audioEnabled: true,
  videoEnabled: true
});

// Listen to events
room.on(RoomEvent.ParticipantJoined, (participant) => {
  console.log(`${participant.name} joined`);
});

room.on(RoomEvent.MessageReceived, (message) => {
  console.log(`${message.senderName}: ${message.content}`);
});

// Media controls
await room.enableVideo();
await room.enableAudio();
await room.shareScreen();

// Send chat message
await room.sendMessage('Hello everyone!');

// Leave meeting
await room.leave();
```

## Components

### MeetingView

A complete meeting experience handling the entire flow from pre-join to meeting room.

```tsx
<MeetingView
  client={client}                    // Required: RQMeetClient instance
  meetingId="meeting_id"             // Required: Meeting ID
  inviteToken="token"                // Recommended: Secure invite token
  initialIdentity="user_123"         // Optional: User identity (legacy)
  isHost={false}                     // Optional: Host flag (legacy, use inviteToken)
  branding={{                        // Optional: Custom branding
    logo: '/logo.png',
    primaryColor: '#6366f1',
    title: 'My Meeting'
  }}
  defaultName="John Doe"             // Optional: Pre-fill display name
  language="en"                      // Optional: 'en' or 'ar'
  direction="ltr"                    // Optional: 'ltr' or 'rtl'
  onLeave={() => {}}                 // Optional: Leave callback
  onMeetingEnded={() => {}}          // Optional: Meeting ended callback
  onDismissed={(reason) => {}}       // Optional: Dismissed from waiting room
  onError={(error) => {}}            // Optional: Error callback
/>
```

### PreJoin

Pre-join screen with device selection, video preview, and waiting room support.

```tsx
<PreJoin
  client={client}                    // Required: RQMeetClient instance
  meetingId="meeting_id"             // Required: Meeting ID
  inviteToken="token"                // Recommended: Secure invite token
  identity="user_123"                // Optional: User identity
  role={ParticipantRole.Participant} // Optional: Participant role
  defaultName="John"                 // Optional: Default display name
  language="en"                      // Optional: 'en' or 'ar'
  direction="ltr"                    // Optional: 'ltr' or 'rtl'
  showVideoPreview={true}            // Optional: Show video preview
  allowDeviceSelection={true}        // Optional: Allow device selection
  branding={{...}}                   // Optional: Custom branding
  onJoin={(data) => {                // Required: Join callback
    // data includes: token, serverUrl, role, permissions, 
    // displayName, audioEnabled, videoEnabled, virtualBackground
  }}
  onDismissed={(reason) => {}}       // Optional: Dismissed callback
  onMeetingEnded={() => {}}          // Optional: Meeting ended callback
  onError={(error) => {}}            // Optional: Error callback
/>
```

### MeetingRoom

Full meeting room experience with video grid, controls, chat, and participant management.

```tsx
<MeetingRoom
  room={room}                        // Required: Connected RQMeetRoom
  client={client}                    // Required: RQMeetClient instance
  meetingId="meeting_id"             // Required: Meeting ID
  role={ParticipantRole.Host}        // Required: Participant role
  permissions={permissions}          // Required: Participant permissions
  inviteToken="token"                // Optional: Invite token
  socket={socketClient}              // Optional: Socket client for real-time
  branding={{...}}                   // Optional: Custom branding
  language="en"                      // Optional: 'en' or 'ar'
  direction="ltr"                    // Optional: 'ltr' or 'rtl'
  enableChat={true}                  // Optional: Enable chat panel
  enableScreenShare={true}           // Optional: Enable screen sharing
  enableRecording={true}             // Optional: Enable recording (hosts)
  onLeave={() => {}}                 // Optional: Leave callback
  onMeetingEnded={() => {}}          // Optional: Meeting ended callback
/>
```

## API Reference

### RQMeetClient

Main client for API interactions.

```typescript
const client = new RQMeetClient({
  apiKey: 'your_api_key',
  apiSecret: 'your_api_secret',      // Optional
  baseUrl: 'https://custom.domain',  // Optional: Override auto-detection
  timeout: 30000,                    // Optional: Request timeout (default: 30000ms)
  headers: { 'X-Custom': 'value' }   // Optional: Custom headers
});

// Note: baseUrl is auto-detected if not provided:
// - http://localhost:3000 when running on localhost
// - https://developer.rqmeet.com for production
```

#### Meeting Management

```typescript
// Create a meeting
const meeting = await client.createMeeting({
  name: 'My Meeting',
  maxParticipants: 50,
  enableRecording: true,
  enableChat: true,
  enableScreenShare: true,
  scheduledStart: new Date('2024-01-01T10:00:00'),
  metadata: { customField: 'value' }
});

// Get meeting details
const meeting = await client.getMeeting('meeting_id');

// List meetings
const response = await client.listMeetings({
  page: 1,
  limit: 10,
  status: MeetingStatus.Active,
  sortBy: 'createdAt',
  sortOrder: 'desc'
});

// End a meeting
await client.endMeeting('meeting_id');

// Update meeting settings
await client.updateMeetingSettings('meeting_id', {
  enableWaitingRoom: true,
  autoAdmit: false
});
```

#### Joining Meetings

```typescript
// Join with full room creation
const room = await client.joinMeeting({
  meetingId: 'meeting_id',
  participantName: 'John Doe',
  participantId: 'user_123',
  isHost: false,
  audioEnabled: true,
  videoEnabled: true
});
```

#### Participant Management

```typescript
// Get participants
const participants = await client.getParticipants('meeting_id');

// Mute a participant
await client.muteParticipant('meeting_id', 'participant_id', true);

// Unmute a participant
await client.muteParticipant('meeting_id', 'participant_id', false);

// Mute all participants
await client.muteAllParticipants('meeting_id');

// Remove a participant
await client.removeParticipant('meeting_id', 'participant_id');
```

#### Recording

```typescript
// Start recording
const recording = await client.startRecording('meeting_id');

// Start with options
const recording = await client.startRecordingWithOptions('meeting_id', {
  layout: 'speaker',
  fileType: 'mp4',
  videoBitrate: 3000000
});

// Stop recording
await client.stopRecording('meeting_id');

// Get recording status
const status = await client.getRecordingStatus('meeting_id');

// List recordings
const recordings = await client.listRecordings('meeting_id');

// Get download URL
const url = await client.getRecordingDownloadUrl('recording_id');

// Delete recording
await client.deleteRecording('recording_id');
```

#### Media Devices

```typescript
// Get available devices
const devices = await client.getMediaDevices();
// devices.audioInputs, devices.audioOutputs, devices.videoInputs

// Request permissions
const stream = await client.requestMediaPermissions(true, true);
```

### RQMeetRoom

Represents an active meeting connection with media controls.

#### Properties

```typescript
room.id                    // Meeting ID
room.roomName              // Room name
room.isConnected           // Connection status
room.connectionState       // ConnectionState enum
room.localParticipant      // Local participant info
room.participants          // Map of remote participants
room.chatMessages          // Array of chat messages
room.virtualBackground     // Current virtual background settings
room.layoutContext         // Layout/focus mode context
```

#### Methods

```typescript
// Connection
await room.connect();
await room.leave();

// Audio controls
await room.enableAudio();
await room.disableAudio();
await room.toggleAudio();
room.isAudioEnabled();

// Video controls
await room.enableVideo();
await room.disableVideo();
await room.toggleVideo();
room.isVideoEnabled();

// Screen sharing
await room.shareScreen();
await room.stopScreenShare();
room.isScreenSharing();

// Virtual background
await room.setVirtualBackground({ type: 'blur', blurRadius: 25 });
await room.setVirtualBackground({ type: 'image', imageUrl: '/bg.jpg' });
await room.setVirtualBackground({ type: 'none' });

// Device selection
await room.setAudioDevice('device_id');
await room.setVideoDevice('device_id');
await room.setAudioOutputDevice('device_id');
const devices = await room.getAvailableDevices();

// Chat
await room.sendMessage('Hello!');

// Focus mode
room.setFocusedTrack(trackReference);
room.clearFocusedTrack();
```

#### Events

```typescript
import { RoomEvent } from '@rqmeet/web-sdk';

room.on(RoomEvent.Connected, () => {});
room.on(RoomEvent.Disconnected, (reason) => {});
room.on(RoomEvent.Reconnecting, () => {});
room.on(RoomEvent.Reconnected, () => {});
room.on(RoomEvent.ParticipantJoined, (participant) => {});
room.on(RoomEvent.ParticipantLeft, (participant) => {});
room.on(RoomEvent.TrackSubscribed, (track, publication, participant) => {});
room.on(RoomEvent.TrackUnsubscribed, (track, publication, participant) => {});
room.on(RoomEvent.TrackMuted, (track, participant) => {});
room.on(RoomEvent.TrackUnmuted, (track, participant) => {});
room.on(RoomEvent.ActiveSpeakersChanged, (speakers) => {});
room.on(RoomEvent.MessageReceived, (message) => {});
room.on(RoomEvent.RecordingStarted, () => {});
room.on(RoomEvent.RecordingStopped, () => {});
room.on(RoomEvent.ConnectionQualityChanged, (quality, participant) => {});
room.on(RoomEvent.MediaDeviceError, (error) => {});
```

### DeveloperSocketClient

Real-time socket client for waiting room and host controls.

```typescript
import { DeveloperSocketClient, SocketEvent } from '@rqmeet/web-sdk';

const socket = new DeveloperSocketClient({
  apiKey: 'your_api_key'
  // baseUrl is auto-detected (same as RQMeetClient)
});

// Connect
await socket.connect();

// Check into meeting with invite token (recommended)
socket.checkWithToken({
  inviteToken: 'invite_token',
  name: 'John Doe'
});

// OR legacy method with identity
socket.check({
  meetingId: 'meeting_id',
  identity: 'user_123',
  name: 'John Doe'
});

// Host: Register to receive waiting room updates
socket.registerHostWithToken({
  meetingId: 'meeting_id',
  inviteToken: 'host_invite_token'
});

// Host: Admit participant from waiting room
socket.allowParticipant({ meetingId: 'meeting_id', identity: 'user_123' });

// Host: Dismiss participant from waiting room
socket.dismissParticipant({ 
  meetingId: 'meeting_id', 
  identity: 'user_123',
  reason: 'Not authorized' 
});

// Host: Mute all participants
socket.muteAll({ meetingId: 'meeting_id' });

// Host: Ask participant to unmute
socket.askToUnmute({ 
  meetingId: 'meeting_id', 
  participantIdentity: 'user_123' 
});

// Host: Update participant role
socket.updateRole({
  meetingId: 'meeting_id',
  identity: 'user_123',
  role: ParticipantRole.CoHost,
  permissions: { canStartRecording: true }
});

// Host: End meeting for all
socket.endMeeting({ meetingId: 'meeting_id' });

// Events
socket.on('connected', () => {});
socket.on('disconnected', (reason) => {});
socket.on('allowed', (data) => {
  // data.token, data.serverUrl, data.role, data.permissions
});
socket.on('dismissed', (data) => {
  // data.reason
});
socket.on('waitingListUpdated', (data) => {
  // data.waitingList: WaitingUser[]
});
socket.on('participantJoined', (data) => {});
socket.on('participantLeft', (data) => {});
socket.on('muteAll', () => {});
socket.on('unmuteRequest', () => {});
socket.on('meetingEnded', () => {});
socket.on('roleUpdated', (data) => {
  // data.role, data.permissions
});
socket.on('error', (error) => {});

// Disconnect
socket.disconnect();
```

## Types

### Enums

```typescript
import {
  MeetingStatus,      // Created, Active, Ended
  ParticipantRole,    // Host, CoHost, Participant
  TrackKind,          // Audio, Video
  TrackSource,        // Camera, Microphone, ScreenShare, ScreenShareAudio
  ConnectionState,    // Disconnected, Connecting, Connected, Reconnecting
  ConnectionQuality,  // Unknown, Poor, Good, Excellent
  RoomEvent,          // All room events
} from '@rqmeet/web-sdk';
```

### Interfaces

```typescript
import type {
  // Client & Config
  RQMeetClientConfig,
  MeetingConfig,
  JoinMeetingOptions,
  
  // Meeting & Participants
  Meeting,
  Participant,
  ParticipantInfo,
  MeetingSettings,
  
  // Tracks & Media
  Track,
  TrackReference,
  MediaDevice,
  VirtualBackground,
  
  // Chat & Recording
  ChatMessage,
  RecordingInfo,
  RecordingOptions,
  
  // Developer Features
  SocketPermissions,
  ExtendedTokenResponse,
  WaitingUser,
  DeveloperMeetingSettings,
  DeveloperMeetingConfig,
  ParticipantRoleDefinition,
  
  // Component Props
  PreJoinProps,
  MeetingRoomProps,
  BrandingConfig,
  
  // Utilities
  LayoutContext,
  UserChoices,
  PaginatedResponse,
} from '@rqmeet/web-sdk';
```

## Error Handling

```typescript
import { RQMeetError, ErrorCode } from '@rqmeet/web-sdk';

try {
  await client.joinMeeting({...});
} catch (error) {
  if (error instanceof RQMeetError) {
    switch (error.code) {
      case ErrorCode.Unauthorized:
        console.log('Invalid API key');
        break;
      case ErrorCode.NotFound:
        console.log('Meeting not found');
        break;
      case ErrorCode.MeetingFull:
        console.log('Meeting is full');
        break;
      case ErrorCode.ConnectionFailed:
        console.log('Failed to connect');
        break;
      default:
        console.log(error.message);
    }
  }
}
```

## Browser Support

- Chrome 74+
- Firefox 78+
- Safari 14.1+
- Edge 79+

## Peer Dependencies

- `react` ^18.0.0
- `react-dom` ^18.0.0

## License

MIT © [RQMeet](https://github.com/rqmeet)
