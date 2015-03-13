# RESTful Web Service
ใน Yii 2 สามารถทำ RESTful ได้ง่ายๆ เลย โดยไม่ต้องติดตั้งเพิ่ม เพราะมีใน v.2 อยู่แล้ว
- สามารถใช้กับ Active Record ได้เลยไม่ต้องเพิ่มเติมอะไร
- ส่งออกได้ทั้ง JSON,XML
- รองรับมาตรฐาน [ATEOAS;](http://en.wikipedia.org/wiki/HATEOAS) ด้วยนะ ไม่ต้องงง! เพราะผมก็งงเหมือนกัน เดี่ยวอธิบายด้านล่าง

ในตัวอย่างนี้ใช้ข้อมูล location พร้อมพิกัด [ดาวโหลดที่นี่](https://raw.githubusercontent.com/bahar/WorldCityLocations/master/World_Cities_Location_table.sql) นำข้อมูลที่ได้ import เข้า mysql ครับ จากนั้นทำการ gii

ต้นฉบับ [Guide REST Quick Start](http://www.yiiframework.com/doc-2.0/guide-rest-quick-start.html) ผมไม่ได้แปลนะ ผมเขียนตามความเข้าใจของผม (ถ้าจะพูดง่ายๆ ผมก็แปลไม่ออกนั่นแหละ T T')
## Create Model
- นำเข้าข้อมูล location  [ดาวโหลดที่นี่](https://raw.githubusercontent.com/bahar/WorldCityLocations/master/World_Cities_Location_table.sql)
- gii สร้าง Model ชื่อ Location

 >  หากยังไม่รู้ว่า gii ทำยังไง [ดูได้ที่นี่](http://www.yiiframework.com/doc-2.0/guide-start-gii.html)

## Create Controller
ทำการสร้าง `LocationController.php` ไว้ที่ `controllers\` โดยที่ตัว controller จะต้อง extends ด้วย ActiveController ซึ่งเป็น class ที่จะทำให้ LocationController เดิมๆ ของเราสามารถใช้งาน RESTful ได้ จากนั้นเราจะต้องเซ็ต overwrite property
- `$modelClass` เพื่อบอกว่า model ที่จะใช้ชื่อว่าอะไร อยู่ที่ใหน อย่าลืม use ด้วยล่ะ ซึ่งตอนนี้เราใช้ `app\models\Locations`
- `serializer`ให้ web service ส่งข้อมูล link หน้าปัจจุบัน,หน้าต่อไป,และหน้าสุดท้ายมาด้วย สะดวกมากๆ
- `collectionEnvelope` เป็นการตั้งชื่อ collection ครอบตัวข้อมูล location ของเราในที่นี้ตั้งชื่อว่า items

> ตอนนี้อาจจะยังไม่เห็นภาพเดี่ยวพอเห็นข้อมูล Json น่าจะเข้าใจ

```php
<?php
namespace app\controllers;

use yii\rest\ActiveController;
use yii\models\Location;

class LocationController extends ActiveController
{
    public $modelClass = 'app\models\Locations';
    public $serializer = [
        'class' => 'yii\rest\Serializer',
        'collectionEnvelope' => 'items',
    ];
}
```
## Configuring URL Rule
เป็นการเปิดการใช้งาน ให้เราสามารถใช้งาน url ในแบบ RESTful ได้ โดยเพิ่มเข้าไปที่ `config\main.php`
ในส่วน components โดยเพิ่มส่วนนี้เข้าไป
>ในส่วนนี้เราต้องเปิดการใช้งาน PrettyUrl ให้ได้เสียก่อน [ดูการติดตั้งได้ที่นี่](https://github.com/dimpled/Yii2-Learning/blob/master/tutorial/modrewrite.md)

ให้เพิ่มคำสั่งนี้เข้าไปที่ `rules` ซึ่งจะเป็นการระบุให้ LocationController  ใช้งาน `url` แบบ RESTful  ได้
```php
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
การใช้งาน เราจะต้องเข้าใจนิยามของการเรียกใช้งาน RESTful ก่อน ซึ่งส่วนใหญ่ก็จะใช้มาตรฐานแบบนี้

| Method | Url | Description |
| ------------- |:-------------:| -----:|
| GET | /location |  แสดงข้อมูล location ทั้งหมด (page by page) |
| HEAD | /location | แสดงข้อมูล location เฉพาะ header (page by page)
| POST | /location | สร้าง locationใหม่ จะต้องแนบข้อมูลมาด้วย
| GET | /location/5 | แสดงข้อมูล location ที่ pk เท่ากับ 5
| HEAD | /location/5 | แสดงข้อมูล location เฉพาะ header ที่ pk เท่ากับ 5
| PATCH | /location/5 | แก้ไขข้อมูล location ที่ pk = 5
| DELETE | /location | ลบข้อมูล locaiont ที่ pk = 5

ทดสอบลองเรียกใช้งานผ่าน `curl` ดู เปิด Terminal แล้วพิมพ์คำสั่งนี้ ส่วน windows ต้องติดต้ง curl เพิ่มหากต้องการทดสอบผ่าน curl [ดาวน์โหลดได้ที่นี่](http://curl.haxx.se/download.html) ขอดีของการทดสอบผ่าน curl คือมันสามารถเรียกได้ ทุก method ถ้ารันผ่าน url บน browser มันจะได้แค่ Method `GET`

> โปรเจคของผมอยู่ที่ `/yii2/yii2-Leanning-Source` ซึ่ง path ของคุณจะไม่ตรงกับของผมซึ่งก็ขึ้นอยู่กับว่าเราตั้งชื่อว่าอะไร


### GET /location
เป็นการเรียกข้อมูล location โดยใช้ method GET ซึ่งจะเป็นการแสดงข้อมูลของ location ทีละ page

Url
```
http://127.0.0.1/yii2/yii2-Leanning-Source/web/location
```

เรียกใช้งาน
```
curl -i -H "Accept:application/json" "http://127.0.0.1/yii2/yii2-Leanning-Source/web/location"
```

>  ลองสังเกตตรง header จะมี header link,X-Pagination ขึ้นมา ซึ่งตรงนี้มีค่า url หน้าปัจจุบัน, หน้าต่อไป, หน้าสุดท้าย ผมเข้าใจว่านี้คือ [ATEOAS;](http://en.wikipedia.org/wiki/HATEOAS) ซื่ง web service ส่วนใหญ่จะมีแค่ data ออกมาเท่านั้นจะไม่มี ATEOAS; ซึ่ง Yii 2 มี  ฮา...ใครมีรายละเอียดก็ช่วยเพิ่มเติมหน่อยละกันครับ


Result
```json
HTTP/1.1 200 OK
Date: Thu, 12 Mar 2015 10:44:50 GMT
Server: Apache/2.4.9 (Unix) PHP/5.6.4
X-Powered-By: PHP/5.6.4
X-Pagination-Total-Count: 10567
X-Pagination-Page-Count: 529
X-Pagination-Current-Page: 1
X-Pagination-Per-Page: 20
Link: <http://127.0.0.1/yii2/yii2-Leanning-Source/web/location/index?page=1>; rel=self, <http://127.0.0.1/yii2/yii2-Leanning-Source/web/location/index?page=2>; rel=next, <http://127.0.0.1/yii2/yii2-Leanning-Source/web/location/index?page=529>; rel=last
Content-Length: 2196
Content-Type: application/json; charset=UTF-8

[{"id":1,"country":"Afghanistan","city":"Kabul","latitude":34.5166667,"longitude":69.1833344,"altitude":1808},{"id":2,"country":"Afghanistan","city":"Kandahar","latitude":31.61,"longitude":65.6999969,"altitude":1015},{"id":3,"country":"Afghanistan","city":"Mazar-e Sharif","latitude":36.7069444,"longitude":67.1122208,"altitude":369},{"id":4,"country":"Afghanistan","city":"Herat","latitude":34.34,"longitude":62.1899986,"altitude":927},{"id":5,"country":"Afghanistan","city":"Jalalabad","latitude":34.42,"longitude":70.4499969,"altitude":573},{"id":6,"country":"Afghanistan","city":"Konduz","latitude":36.72,"longitude":68.8600006,"altitude":394},{"id":7,"country":"Afghanistan","city":"Ghazni","latitude":33.5535554,"longitude":68.4268875,"altitude":2175},{"id":8,"country":"Afghanistan","city":"Balkh","latitude":36.7586111,"longitude":66.8961105,"altitude":328},{"id":9,"country":"Afghanistan","city":"Baghlan","latitude":36.12,"longitude":68.6999969,"altitude":565},{"id":10,"country":"Afghanistan","city":"Gardez","latitude":33.59,"longitude":69.2200012,"altitude":2279},{"id":11,"country":"Afghanistan","city":"Khost","latitude":33.3380556,"longitude":69.9202805,"altitude":1178},{"id":12,"country":"Afghanistan","city":"Khanabad","latitude":36.68,"longitude":69.1100006,"altitude":490},{"id":13,"country":"Afghanistan","city":"Tashqorghan","latitude":36.6952778,"longitude":67.6980591,"altitude":460},{"id":14,"country":"Afghanistan","city":"Taloqan","latitude":36.7360511,"longitude":69.5345078,"altitude":788},{"id":15,"country":"Afghanistan","city":"Cool urhajo","latitude":34.2654452,"longitude":67.3451614,"altitude":2733},{"id":16,"country":"Afghanistan","city":"Pol-e Khomri","latitude":35.9425,"longitude":68.7191696,"altitude":832},{"id":17,"country":"Afghanistan","city":"Sheberghan","latitude":36.6672222,"longitude":65.7536087,"altitude":362},{"id":18,"country":"Afghanistan","city":"Charikar","latitude":35.013611,"longitude":69.1713867,"altitude":1534},{"id":19,"country":"Afghanistan","city":"Sar-e Pol","latitude":36.2155556,"longitude":65.9363861,"altitude":629},{"id":20,"country":"Afghanistan","city":"Paghman","latitude":34.5875,"longitude":68.953331,"altitude":2276}]%
```

### GET /location/5
เป็นการเรียกข้อมูล location ที่มี primary key  เท่ากับ 5
url
```
http://127.0.0.1/yii2/yii2-Leanning-Source/web/location/5
```
เรียกใช้งาน
```
curl -i -H "Accept:application/json" "http://127.0.0.1/yii2/yii2-Leanning-Source/web/location/5"
```
Result
```json
HTTP/1.1 200 OK
Date: Thu, 12 Mar 2015 17:54:52 GMT
Server: Apache/2.4.9 (Unix) PHP/5.6.4
X-Powered-By: PHP/5.6.4
Content-Length: 106
Content-Type: application/json; charset=UTF-8

{"id":5,"country":"Afghanistan","city":"Jalalabad","latitude":34.42,"longitude":70.4499969,"altitude":573}%
```


> สำหรับคนที่ไม่ชอบ geek ก็ใข้แบบ Gui ได้ครับเป็น extension ของ Chrome Browser [ดาวน์โหลด](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm) ^^ หลายคนอาจจะนึกในใจว่า ทำไมไม่บอกตั้งแต่แรก....! เอ่อผมอยากให้ททุกคน geek ครับ ฮา สวัสดีครับ...

ดูสิใช้งานง่ายมากเลย ...
![postman](/images/postman.png)
