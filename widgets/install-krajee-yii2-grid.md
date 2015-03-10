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

### Config

ในตัวอย่างนี้ผมได้สร้างตาราง `countries` ไว้แล้วและทำการ gii crud ไว้เรียบร้อย
ซึ่งเราจะทำการเปลี่ยนการใช้งานจากของเดิม ` yii\grid\GridView` เป็น `kartik\grid\GridView;` โดยแก้ไขที่ไฟล์ `countries/index.php`
จากเดิม
```
use yii\grid\GridView;
```
เปลี่ยนเป็น
```
use kartik\grid\GridView;
```

จากนั้นในส่วนของการเรียกใช้งาน `GridView::widget([....])` เพิ่มค่าคอนฟิก properties `panel` เข้าไป
```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    //....
    'panel'=>[
        'before'=>''
    ],
    //.....
    )];
?>
```

ลองทำการสดสอบรันดู จะพบ `export menu` ที่ด้านขวาให้เราสามารถคลิกแล้วเลือก  ตามที่เราต้องการได้โดยมีรายการให้เลือกดังนี้
- Html
- CSV
- Text
- Excel
- Pdf
- Json
