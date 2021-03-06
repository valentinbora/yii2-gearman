yii2-gearman
============

This extension built on [this](https://github.com/Filsh/yii2-gearman) and [this](https://github.com/sinergi/gearman).
The goal of the project is opportunity of starting multiple worker processes on one machine. 

## Installation

It is recommended that you install the Gearman library [through composer](http://getcomposer.org/). To do so, add the following lines to your ``composer.json`` file.

```json
{
    "repositories": [
        {
            "url": "https://github.com/valentinbora/yii2-gearman.git",
            "type": "git"
        }
    ],
    "require": {
       "valentinbora/yii2-gearman": "dev-master"
    }
}
```

## Configuration

```php
'components' => [
  'gearman' => [
      'class' => 'shakura\yii2\gearman\GearmanComponent',
      'servers' => [
          ['host' => '127.0.0.1', 'port' => 4730],
      ],
      'user' => 'www-data',
      'jobs' => [
          'syncCalendar' => [
              'class' => 'common\jobs\SyncCalendar'
          ],
          ...
      ]
  ]
],
...
'controllerMap' => [
    'gearman' => [
        'class' => 'shakura\yii2\gearman\GearmanController',
        'gearmanComponent' => 'gearman'
    ],
    ...
],
```

## Job example

```php
namespace common\jobs;

use shakura\yii2\gearman\JobBase;

class SyncCalendar extends JobBase
{
    public function execute(\GearmanJob $job = null)
    {
        // Do something
    }
}
```

## Manage workers

```cmd
yii gearman/start 1 [jobsFilter] // start the worker with unique id
yii gearman/start 1 jobName,jobName2 // start the worker with unique id and whitelist two jobs
yii gearman/start 1 jobName,not:jobName2 // start the worker with unique id, whitelist jobName and blacklist jobName2
yii gearman/start 1 all,not:jobName2 // start the worker with unique id, whitelist all jobs but blacklist jobName2
yii gearman/restart 1 // restart worker
yii gearman/stop 1 // stop worker
```

The jobsFilter optional parameter is a string of comma separated job names that you want to filter. If you don't specify it, all jobs available in the system will be available to the worker being started.

The jobsFilter parameter also accepts a negation operator `not:` as a prefix to the job name, in order to blacklist that job type.

## Example using Dispatcher

```php
Yii::$app->gearman->getDispatcher()->background('syncCalendar', new JobWorkload([
    'params' => [
        'data' => 'value'
    ]
])); // run in background
Yii::$app->gearman->getDispatcher()->execute('syncCalendar', new JobWorkload([
    'params' => [
        'data' => 'value'
    ]
])); // run synchronize
```

## Example of [Supervisor](http://supervisord.org/) config to manage multiple workers

```
[program:yii-gearman-worker]
command=php [path_to_your_app]/yii gearman/start %(process_num)s
process_name=gearman-worker-%(process_num)s
priority=1
numprocs=5
numprocs_start=1
autorestart=true
```
