# RESTful Web Service
ใน Yii 2 สามารถทำ RESTful ได้ง่ายๆ เลย โดยไม่ต้องติดตั้งเพิ่ม เพราะมีใน v.2 อยู่แล้ว
- สามารถใช้กับ Active Record ได้เลยไม่ต้องเพิ่มเติมอะไร
- ส่งออกได้ทั้ง JSON,XML

ในตัวอย่างนี้ใช้ข้อมูล location พร้อมพิกัด [ดาวโหลดที่นี่](https://raw.githubusercontent.com/bahar/WorldCityLocations/master/World_Cities_Location_table.sql) นำข้อมูลที่ได้ import เข้า mysql ครับ จากนั้นทำการ gii สร้าง model ชื่อ Location

## Create Controller
ทำการสร้าง `LocationController.php` ไว้ที่ `controllers\` โดยที่ตัว controller จะต้อง extends ด้วย ActiveController ซึ่งเป็น class ที่จะทำให้ LocationController เดิมๆ ของเราสามารถใช้งาน RESTful ได้ จากนั้นเราจะต้องเซ็ต overwrite property `$modelClass` เพื่อบอกว่า model ที่จะใช้ชื่อว่าอะไร อยู่ที่ใหน ซึ่งตอนนี้เราใช้ 'app\models\Locations'
```php
<?php
namespace app\controllers;

use yii\rest\ActiveController;

class UserController extends ActiveController
{
    public $modelClass = 'app\models\Locations';
}
```
## Configuring URL Rule
เป็นการเปิดการใช้งาน ให้เราสามารถใช้งาน url ในแบบ RESTful ได้ โดยเพิ่มเข้าไปที่ `config\main.php`
ในส่่วน components โดยเพิ่มส่วนนี้เข้าไป
>ในส่วนนี้เราต้องเปิดการใช้งาน PrettyUrl ให้ได้เสียก่อน [ดูการติดตั้งได้ที่นี่](https://github.com/dimpled/Yii2-Learning/blob/master/tutorial/modrewrite.md)

ให้เพิ่มคำสั่งนี้เข้าไปที่ `rules` ซึ่งจะเป็นการระบุให้ LocationController  ใช้งาน `url` แบบ RESTful  ได้
```
['class' => 'yii\rest\UrlRule', 'controller' => 'location'],
```
จะได้แบบนี้
```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'enableStrictParsing' => true,
    'showScriptName' => false,
    'rules' => [
        ['class' => 'yii\rest\UrlRule', 'controller' => 'location'],
    ],
]
```

หากใครเปิดการใช้งาน PrettyUrl อยู่แล้วก็เพิ่มเข้าไปแบบนี้
```php
'urlManager' => [
  'class' => 'yii\web\UrlManager',
  'showScriptName' => false,
  'enablePrettyUrl' => true,
  'rules' => [
        '<controller:\w+>/<id:\d+>' => '<controller>/view',
        '<controller:\w+>/<action:\w+>/<id:\d+>' => '<controller>/<action>',
        '<controller:\w+>/<action:\w+>' => '<controller>/<action>',
        ['class' => 'yii\rest\UrlRule', 'controller' => 'location'],
  ],
],

```

## Enabling JSON Input
เปิดการเปิดการใช้งาน ให้สามารถรับค่าที่อยู่ในรูปแบบของ Json ได้ เพราะเวลาที่เราเรียกใช้งานหรือทำการส่งค่าอะไรมามันจะต้องอยู่ในรูปแบบของ Json โดยเพิ่มเข้าไปที่ application components ของ config/main.php  ใส่โค้ดเข้าไปดังนี้

ของเดิมจะเป็นแบบนี้และส่วนของ `cookieValidationKey` ค่าจะไม่เหมือนกัน
```php
'request' => [
    'cookieValidationKey' => 'xxxxx......',
],
```
โดยเพิ่มเข้าไป
```php
'request' => [
    'cookieValidationKey' => 'xxxxx......',
    'parsers' => [
        'application/json' => 'yii\web\JsonParser',
    ]
]
```

## Trying it Out
การใช้งาน เราจะต้องเข้าใจนิยามของการเรียกใช้งาน RESTful ก่อนซึี่งจะมีรูปแบบที่ส่วนใหญ่มีมาตรฐานแบบนี้คือ
