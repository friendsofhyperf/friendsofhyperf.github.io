# Lock

Hyperf atomic lock component.

## Installation

- Install

```shell
composer require friendsofhyperf/lock
```

- Publish Configuration

```shell
php bin/hyperf.php vendor:publish friendsofhyperf/lock -i config
```

## Usage

You can use the `lock()` method to create and manage locks:

```php
$lock = lock($name = 'foo', $seconds = 10, $owner = null);

if ($lock->get()) {
    // Lock acquired for 10 seconds...

    $lock->release();
}
```

The `get` method can also accept a closure. After the closure executes, the lock will be automatically released:

```php
lock('foo')->get(function () {
    // Lock acquired indefinitely and automatically released...
});
```

If the lock is unavailable during the request, you can control waiting for a specified number of seconds. If the lock cannot be acquired within the specified time limit, a `FriendsOfHyperf\Lock\Exception\LockTimeoutException` will be thrown:

```php
use FriendsOfHyperf\Lock\Exception\LockTimeoutException;

$lock = lock('foo', 10);

try {
    $lock->block(5);

    // Lock acquired after waiting up to 5 seconds...
} catch (LockTimeoutException $e) {
    // Unable to acquire lock...
} finally {
    optional($lock)->release();
}

lock('foo', 10)->block(5, function () {
    // Lock acquired after waiting up to 5 seconds...
});
```

Using Annotations

```php
use FriendsOfHyperf\Lock\Annotation\Lock;
use FriendsOfHyperf\Lock\Driver\LockInterface;

class Foo
{
    #[Lock(name:"foo", seconds:10)]
    protected LockInterface $lock;

    public function bar()
    {
        $this->lock->get(function () {
            // Lock acquired indefinitely and automatically released...
        });
    }
}
```
