## Scenario

1. 触发事件，然后「立即通过」监听器去执行后续的动作（观察者模式）
2. 触发事件，然后「投递队列」再「通过消费队列」执行后续动作（生产-消费模式）
3. LARAVEL 还设计了一个「事件订阅者」`从订阅者类本身中订阅多个事件的类，允许你在单个类中定义多个事件处理程序`


## Usage

1. 手动注册
```
protected $listen = [
\App\Events\AdvisorCreated::class => [
    \App\Listeners\AdvisorCreated\SendToSalesforce::class,
    \App\Listeners\AdvisorCreated\CreateSampleHouseholds::class,
],
```
2. Callback
```
Event::listen(['eloquent.saving: *', 'eloquent.updating: *', 'eloquent.creating: *', 'eloquent.deleting: *', 'eloquent.restoring: *', 'eloquent.forceDeleting: *'], function (string $event_name, array $data): bool|null {
    /** @var \Illuminate\Database\Eloquent\Model $object */
    $object = $data[0];

    if ($object instanceof Model) {
        $event_name = strstr_or_fail(substr($event_name, 9), ':', true); // remove 'eloquent.' prefix and ': xxxxx' postfix

        // Call event functions
        $fqcn = Model::NAMESPACE . '\Events\\' . class_basename($object) . 'Event';

        if (class_exists($fqcn)) {
            if (method_exists($fqcn, $event_name) || $object instanceof Tweak) { // Call TweakEvent::__callStatic($event_name) if event method is not exist
                return $fqcn::$event_name($object);
            }
        }
    }

    return null;
});
```
3. Event Subscriber
4. Auto discover
```
/**
 * 确定是否应用自动发现事件和监听器。
 
 * DEFAUT FALSE
 */
public function shouldDiscoverEvents(): bool
{
    return true;
}
```
