# Model Scope

Hyperf 框架的模型 Scope 註解元件。

## 安裝

```shell
composer require friendsofhyperf/model-scope
```

## 使用

- 定義 Scope

```php
namespace App\Model\Scope;

use Hyperf\Database\Model\Builder;
use Hyperf\Database\Model\Model;
use Hyperf\Database\Model\Scope;
 
class AncientScope implements Scope
{
    /**
     * Apply the scope to a given Model query builder.
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->subYears(2000));
    }
}
```

- 繫結到模型

```php
namespace App\Model;
 
use App\Model\Scope\AncientScope;
use FriendsOfHyperf\ModelScope\Annotation\ScopedBy;
 
#[ScopedBy(AncientScope::class)]
class User extends Model
{
    //
}
```
