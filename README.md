# Laravel Eloquent Filter

## Installation
Minimum version of PHP **7.2**\
Require this package with composer.

```
composer require exenjer/laravel-filters
```
## Usage
```php
public function index(Request $request, UserFilter $userFilter)
{
    $users = $userFilter->filter($request->all())
        ->get();
    // OR
    $users = $userFilter->filter($request->all())
        ->paginate();
    // OR
    $users = $userFilter->filter($request->all())
        ->simplePaginate();
        
    // OR
    $users = (new UserFilter())->filter($request->all())
        ->get();
    // ...
}
```

## Configuration
### Basic configuration
Create a new class and extend from `Filter`
```php
class UserFilter extends Filter
{

}
```

Add filter method:
```php
class UserFilter extends Filter
{
    public function filter(array $request): Filter
    {
        $this->setFilter($this); // Set this class as filter
    
        return $this->apply($request); // Apply filter
    }
}
```
or use trait for default filter method:
```php
class UserFilter extends Filter
{
    use DefaultFilterSetUp;
}
```
Specify the model with which the filter will work:
```php
class UserFilter extends Filter
{
    use DefaultFilterSetUp;
    
    protected $model = User::class;
}
```
Specify the fields to which the filtering will react:
```php
class UserFilter extends Filter
{
    use DefaultFilterSetUp;
    
    protected $model = User::class;
    
    protected $fields = ['name', 'age'];
}
```
You are ready to write filters!

The following scheme is used for writing filters: Create a method with the following name: **'fieldName' + 'Filter'** word for handle all request 
values except array and **'fieldName' + 'ArrayFilter'** for handle arrays.
>You may not create methods to filter specific fields. Then the standard methods will be called, details of which are written below.

*Example*:
```php
class UserFilter extends Filter
{
    use DefaultFilterSetUp;
    
    protected $model = User::class;
    
    protected $fields = ['name', 'age'];
    
    protected function nameFilter(string $value): void
    {
        $this()->where('name', $value);
    }

    protected function nameArrayFilter(array $value): void
    {
        $this()->whereIn('name', $value);
    }
}
```
Yes, in order to gain access to the Builder, you need to refer to `$this()`
### Additional configuration
For using all filters `withTrashed`, set true for `$withDeletions` property:
```php
protected $withDeletions = true;
```

Also, you can cast a value for the resulting values via `$casts` property:

> Does not work when getting an array
```php
protected $casts = [
    'age' => 'int'
];
```
#### Dynamic filters
You can dynamically add filters in two ways:
1. Using the method `addFilter()`
2. Using the interface `FilterRule` and method `addFilterClass()`

An example implementation of the first method (in UserFilter context):
```php
$this->addFilter('name', function (string $name): void {
    $this()->where('name', $name);
}, function (array $values): void { // For arrays
    $this()->whereIn('name', $values);
});
```
The implementation of the second method:

Create a class and implement `FilterRule` interface
```php
class Test2UserFilterRule implements FilterRule
{
    public function handle($value, Builder $builder): void
    {
        $builder->where('name', $value);
    }
 
    public function arrayHandle(array $values): void
    {
        $this()->whereIn('name', $values);
    }
}
```