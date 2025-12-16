---
layout: post
title: Implementing Redis Pub/Sub for Real-Time Notifications in PHP Applications on LAMPP (XAMPP for Linux)
---

## Introduction

In today's web development world, real-time notifications (Real-Time Notifications) are one of the most important features for creating an engaging user experience. Redis, with its lightweight and ultra-fast Publish/Subscribe (Pub/Sub) system, is one of the best options for implementing this capability in a local development environment like **LAMPP** (i.e., XAMPP on Linux/Ubuntu).

This guide explains step-by-step and completely practically how to set up and use Redis Pub/Sub in PHP applications running on **LAMPP**.

> Important Note: Redis Pub/Sub does not store messages. If the subscriber is not active at the time of publishing, the message is lost. For stable needs, use Redis Lists, Streams, or other message queues alongside Pub/Sub.

## Prerequisites

- Operating System **Ubuntu** (or any Debian-based distribution)
- **LAMPP** (XAMPP for Linux) installed (default path: `/opt/lampp`)
- Redis Server
- PHP-Redis extension (phpredis) compatible with LAMPP's PHP version
- Root or sudo access
- Multiple terminals for simultaneous testing of publisher and subscriber

## Installing and Setting Up Redis

```markdown
sudo apt update
sudo apt install redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

Check status:
```markdown
redis-cli ping
# Should return PONG
```

## Installing phpredis Extension for LAMPP's PHP Version

**Warning**: The `php-redis` package from Ubuntu repositories is not compatible with LAMPP's custom PHP. Be sure to use PECL:

```markdown
# Install necessary tools for compilation
sudo apt install build-essential autoconf

# Use pecl inside LAMPP
sudo /opt/lampp/bin/pecl install redis
```

If prompted, accept the default options.

Then add the following line to the `/opt/lampp/etc/php.ini` file:

```markdown
extension=redis.so
```

Restart LAMPP:

```markdown
sudo /opt/lampp/lampp restart
```

Check successful installation:
```markdown
/opt/lampp/bin/php -m | grep redis
# Should display redis
```

If for any reason PECL doesn't work, you can use the Predis library (Pure PHP):

```markdown
composer require predis/predis
```

## Initial Connection Test

Create the file `test_redis.php`:

```markdown
<?php
$redis = new Redis();
$redis->pconnect('127.0.0.1', 6379);
echo $redis->ping() === '+PONG' ? 'Connection successful!' : 'Connection error';
?>
```

Run:

```markdown
/opt/lampp/bin/php test_redis.php
```

## Implementing Publisher (Sender)

```markdown
<?php
// publisher.php
$redis = new Redis();
$redis->pconnect('127.0.0.1', 6379);
// $redis->auth('yourpassword'); // If password is set

$channel = 'notifications';
$message = json_encode([
    'type'    => 'new_message',
    'user_id' => 123,
    'title'   => 'New message',
    'body'    => 'Hello, real-time world!'
], JSON_UNESCAPED_UNICODE);

$sent = $redis->publish($channel, $message);
echo "Message sent ($sent subscribers received it)\n";
?>
```

## Implementing Subscriber (Receiver)

```markdown
<?php
// subscriber.php
$redis = new Redis();
$redis->pconnect('127.0.0.1', 6379);
$redis->setOption(Redis::OPT_READ_TIMEOUT, -1); // Permanent blocking

$callback = function ($redis, $channel, $message) {
    try {
        $data = json_decode($message, true, 512, JSON_THROW_ON_ERROR);
        echo "Received from channel [$channel]:\n";
        print_r($data);
        echo "────────────────────\n";

        // Here you can:
        // - Send desktop notification
        // - Push to WebSocket
        // - Store in database
        // - Send email and ...
    } catch (Throwable $e) {
        error_log("JSON decode error: " . $e->getMessage());
    }
};

echo "Listening to channel 'notifications' ...\n";
$redis->subscribe(['notifications'], $callback);
?>
```

Running subscriber in the background:

```markdown
nohup /opt/lampp/bin/php /path/to/subscriber.php > /dev/null 2>&1 &
```

Or better yet with **Supervisor** (final recommendation):

```markdown
sudo apt install supervisor
```

Configuration file:

```markdown
# /etc/supervisor/conf.d/redis-notifications.conf
[program:redis-notifications]
command=/opt/lampp/bin/php /path/to/your/project/subscriber.php
directory=/path/to/your/project
autostart=true
autorestart=true
user=your-username
stdout_logfile=/var/log/redis-notifications.out.log
stderr_logfile=/var/log/redis-notifications.err.log
```

Apply changes:

```markdown
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start redis-notifications
```

## Integration into LAMPP Application (Real Example)

```markdown
// send_notification.php (in your app's htdocs)
function sendNotification(int $userId, string $title, string $body): void
{
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);

    $payload = json_encode([
        'user_id' => $userId,
        'title'   => $title,
        'body'    => $body,
        'time'    => date('c')
    ], JSON_UNESCAPED_UNICODE);

    $redis->publish("notifications.user{$userId}", $payload);
    $redis->publish('notifications.all', $payload); // For all users
}

// Example usage after registering a new message in the database
sendNotification(123, 'New message', 'User Ali sent you a message');
```

## Security Tips and Optimization for LAMPP

1. Bind Redis only to localhost:
   ```markdown
   # /etc/redis/redis.conf
   bind 127.0.0.1
   requirepass YourStrongPasswordHere
   ```

2. Disable dangerous commands:
   ```markdown
   rename-command CONFIG ""
   rename-command FLUSHALL ""
   rename-command FLUSHDB ""
   ```

3. Always use `pconnect()` instead of `connect()` (stable connection).

4. For large projects, use **Swoole**, **ReactPHP**, or **RoadRunner** instead of CLI PHP.

## Delivering Notifications to the Browser (Frontend)

Common methods:
- Ratchet (Pure PHP WebSocket) + Redis subscriber
- Node.js + Socket.IO + Redis Adapter
- Server-Sent Events (SSE) with PHP
- Long polling (only for testing)

## Conclusion

By using Redis Pub/Sub in the **LAMPP** environment on Ubuntu, you can easily build a powerful real-time notification system without the need for constant polling. This solution:

- Is extremely fast
- Consumes little memory
- Works seamlessly with LAMPP's PHP version
- Is completely suitable for local development and testing environment

Now you can implement chat, management dashboard, order notifications, real-time likes, and hundreds of other features with the least cost.

Good luck and enjoy coding!