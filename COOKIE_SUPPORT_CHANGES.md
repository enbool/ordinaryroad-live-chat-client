# Cookie Support in Connect Method - Implementation Summary

## Overview
Added Cookie header support to the WebSocket connection handshake for all live chat clients. When calling the `connect()` method, any cookie configured in the client config will now be automatically sent in the WebSocket handshake request headers.

## Changes Made

### 1. Kuaishou Client
**File**: `live-chat-clients/live-chat-client-kuaishou/src/main/java/tech/ordinaryroad/live/chat/client/kuaishou/client/KuaishouLiveChatClient.java`

Modified the `initConnectionHandler()` method to:
- Create a `DefaultHttpHeaders` object
- Check if cookie is configured via `getConfig().getCookie()`
- Add the Cookie header to the WebSocket handshake if present

### 2. Huya Client
**File**: `live-chat-clients/live-chat-client-huya/src/main/java/tech/ordinaryroad/live/chat/client/huya/client/HuyaLiveChatClient.java`

Modified the `initConnectionHandler()` method to:
- Create a `DefaultHttpHeaders` object
- Check if cookie is configured via `getConfig().getCookie()`
- Add the Cookie header to the WebSocket handshake if present

### 3. Bilibili Client
**File**: `live-chat-clients/live-chat-client-bilibili/src/main/java/tech/ordinaryroad/live/chat/client/bilibili/client/BilibiliLiveChatClient.java`

Modified the `initConnectionHandler()` method to:
- Add cookie check to the existing headers object
- Append the Cookie header to existing headers if configured

### 4. Douyin Client
**File**: `live-chat-clients/live-chat-client-douyin/src/main/java/tech/ordinaryroad/live/chat/client/douyin/client/DouyinLiveChatClient.java`

Modified the `initConnectionHandler()` method to:
- Check if additional cookie is configured
- Merge the additional cookie with the existing ttwid cookie
- Properly concatenate multiple cookie values with "; " separator

### 5. Douyu Client
**File**: `live-chat-clients/live-chat-client-douyu/src/main/java/tech/ordinaryroad/live/chat/client/douyu/client/DouyuWsLiveChatClient.java`

Modified the `initConnectionHandler()` method to:
- Create a `DefaultHttpHeaders` object
- Check if cookie is configured via `getConfig().getCookie()`
- Add the Cookie header to the WebSocket handshake if present

### 6. WebSocket Client
**File**: `live-chat-clients/live-chat-client-websocket/src/main/java/tech/ordinaryroad/live/chat/client/websocket/client/WebSocketLiveChatClient.java`

Modified the `initConnectionHandler()` method to:
- Create a `DefaultHttpHeaders` object
- Check if cookie is configured via `getConfig().getCookie()`
- Add the Cookie header to the WebSocket handshake if present

## Usage Example

```java
// Example: Kuaishou Client with Cookie
KuaishouLiveChatClientConfig config = KuaishouLiveChatClientConfig.builder()
    .roomId(123456)
    .cookie("your_cookie_string_here")
    .build();

KuaishouLiveChatClient client = new KuaishouLiveChatClient(config, msgListener);
client.connect();  // Cookie will be automatically included in WebSocket handshake
```

```java
// Example: Bilibili Client with Cookie
BilibiliLiveChatClientConfig config = BilibiliLiveChatClientConfig.builder()
    .roomId(123456)
    .cookie("SESSDATA=xxx; bili_jct=yyy")
    .build();

BilibiliLiveChatClient client = new BilibiliLiveChatClient(config, msgListener);
client.connect();  // Cookie will be automatically included in WebSocket handshake
```

## Technical Details

### Implementation Pattern
All clients now follow this pattern in their `initConnectionHandler()` method:

```java
DefaultHttpHeaders customHeaders = new DefaultHttpHeaders();
// ... add other headers as needed ...

// Add Cookie to request headers
if (StrUtil.isNotBlank(getConfig().getCookie())) {
    customHeaders.add("Cookie", getConfig().getCookie());
}

return new ConnectionHandler(
    () -> new WebSocketClientProtocolHandler(
        WebSocketClientProtocolConfig.newBuilder()
            .customHeaders(customHeaders)  // Use customHeaders instead of new DefaultHttpHeaders()
            // ... other config ...
            .build()
    ),
    // ... other parameters ...
);
```

### Special Case: Douyin Client
The Douyin client has special handling because it already sets a ttwid cookie. The implementation:
1. Always adds the ttwid cookie first
2. If additional cookie is configured, it merges them with "; " separator
3. This allows for multiple cookie values in a single Cookie header

## Testing
All changes have been verified to compile without errors. The implementations:
- Do not break existing functionality
- Are backward compatible (cookie is optional)
- Follow the existing code patterns in each client
- Use proper null/blank checks before adding cookies

## Notes
- Cookie values should be provided in standard HTTP Cookie header format
- Multiple cookies can be separated by "; " (semicolon and space)
- The cookie is sent during the WebSocket handshake, not in subsequent messages
- Each client's configuration object must have a `cookie` field for this to work (already present in base config)

