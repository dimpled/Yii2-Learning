# การติดตั้ง GridView + ExportMenu

 ในตัวอย่างนี้เป็นการติดตั้งและใช้งาน widget ของ `kartik-v/yii2-grid` โดยที่ตัว GridView ตัวนี้มีความสามารถเยอะมากๆ แต่หลักๆ ที่ผมชอบเลยก็คือ สามารถ export ข้อมูลได้หลายแบบ เช่น .xls, .pdf, .csv, .txt, .html ซึ่งสามารถคลิกเลือกจากตัว GridView ได้โดยตรงเลย

### Install
ในการใช้งาน export menu จะต้องติดตั้ง  ` widget`ทั้งหมด 3 ตัว โดยเพิ่มที่ไฟล์ composer.json ในส่วนของ  ```require```

 ```
"kartik-v/yii2-grid": "*",
"kartik-v/yii2-export": "*",
"kartik-v/yii2-mpdf": "*"
 ```

จากนั้นทำการสั่งติดตั้งโดยใช้คำสั่ง
```
composer update
```
![install grid](/images/install-grid.png "Logo Title Text 1")

### Config
ตัวอย่างนี้จะใช้ข้อมูลจากตาราง countries
> ดูโครงสร้างตาราง `countries` [countries.sql](https://github.com/raramuridesign/mysql-country-list/blob/master/mysql-country-list.sql) ได้ที่นี่

หลังจากสร้างตาราง countries ใน mysql แล้วให้ gii countries Model และ gii countries CRUD

จากนั้นเพิ่ม module gridview ใน `config\web.php` ตามด้านล่าง ตัวนี้เป็น module ที่ใช้ในการ export  (csv, text, html, or xls)
เพื่อรองรับการ export ที่เราคลิกเลือกจากเมนู
```php
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'bootstrap' => ['log'],
    //...........
    'modules' => [
       'gridview' =>  [
            'class' => '\kartik\grid\Module'
        ]
    ],
```

จากนั้นไปที่ `views\countries\index.php` ทำการเปลี่ยนการเรียกใช้งานในส่วน `use` Gridview
จากเดิม

```php
<?php
use yii\grid\GridView;
```
เปลี่ยนเป็น
```php
<?php
use kartik\grid\GridView;
```

และในส่วนของการเรียกใช้งาน `GridView::widget([....])` เพิ่มค่าคอนฟิก properties `panel` เข้าไป
```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    //....
    'panel'=>[
        'before'=>''
    ],
    // ...
    'columns' => [
        'id',
        'name',
        'created_at:datetime'
    ],
]) ?>
```

สดสอบรันดู จะพบไอคอน ที่ด้านขวาให้เราสามารถคลิกแล้วเลือก   export ตามที่เราต้องการได้โดยมีรายการให้เลือกดังนี้
- Html
- CSV
- Text
- Excel
- Pdf
- Json
