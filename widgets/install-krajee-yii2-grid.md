# การติดตั้ง GridView + ExportMenu

 ในตัวอย่างนี้เป็นการติดตั้งและใช้งาน widget ของ [kartik-v/yii2-grid](https://github.com/kartik-v/yii2-grid/blob/master/README.md) โดยที่ตัว GridView ตัวนี้มีความสามารถเยอะมากๆ แต่หลักๆ ที่ผมชอบเลยก็คือ สามารถ export ข้อมูลได้หลายแบบ เช่น .xls, .pdf, .csv, .txt, .html ซึ่งสามารถคลิกเลือกจากตัว GridView ได้โดยตรงเลย

 มี 2 แบบคือ
 * Export from GridView
 * Export Menu


## Export from GridView

![index](/images/index.png)

### Install
ในการใช้งาน export menu จะต้องติดตั้ง  ` widget`ทั้งหมด 4 ตัว โดยเพิ่มที่ไฟล์ composer.json ในส่วนของ  ```require```

 ```
"kartik-v/yii2-widgets": "*",
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

และในส่วนของการเรียกใช้งาน `GridView::widget([....])` เพิ่มค่าคอนฟิก properties `panel` เข้าไป และปิด ActionColumn ไว้เพราะเวลา export file เดี่ยวเมนูจะตามไปด้วย
```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    //........
    'panel'=>[
            'before'=>' '
    ],
    //.........
    'columns' => [
        ['class' => 'yii\grid\SerialColumn'],
        'id',
        'country_code',
        'country_name',
        //['class' => 'yii\grid\ActionColumn'],
    ],
]); ?>
```

ทดสอบรันดู จะพบไอคอน ที่ด้านขวาให้เราสามารถคลิกแล้วเลือก   export ตามที่เราต้องการได้โดยมีรายการให้เลือกดังนี้
- Html
- CSV
- Text
- Excel
- Pdf
- Json

![export menu](/images/grid-export.png)

ลองทำการทดลอง export เลือก format ที่ต้องการ ระบบจะบอกว่ามันจะ generated file สำหรับดาวน์โหลดให้ คุณต้องการทำหรือปล่าว ให้ตอบ OK
> หากไม่ขึ้น dialog ลองเช็คว่ามีการปิด popup ใน browser หรือไม่

![export1](/images/export1.png)
ระบบจะ export file ให้รอซักครู่
![export2](/images/export2.png)
หลังจากเสร็จเราสามารถเปิดดูไฟล์ได้เลย
![export3](/images/export3.png)

## ตัวอย่างไฟล์ที่ export
### Html
![html](/images/html.png)
### Text
![text](/images/txt.png)
### CSV
![csv](/images/csv.png)
### Pdf
![pdf](/images/pdf.png)
### Excel
ผมลองแล้วยัง error! ไม่แนใจเหมือนกันกับ `Gridview`ตัวนี้ แต่สามารถใช้ตัว export menu ได้
### Json
![json](/images/json.png)

<br>
<br>
## Export Menu

![](/images/exportmenu.png)

ตัวนี้เราได้ติดตั้งไปแล้วสามารถเรียกใช้ได้เลย export menu จะรับค่า DataProvider เหมือนกับ `GridView`

ไปที่ `views\countries\index.php` เพิ่ม widgit เข้าไป
> ตัวนี้สามารถ export excel ได้ทั้ง 2 แบบคือ .xls, .xlsx

```php
<?php
use kartik\export\ExportMenu;
echo ExportMenu::widget([
    'dataProvider' => $dataProvider
]);?>
```
หลังจากนั้นก็สามารถเรียกใช้งานได้เลย ง่ายมากๆ ครับ


## การติดตั้งภาษาไทยให้กับ MPDF
> วิธินี้อาจไม่ใช่วิธีที่ถูกเพราะถ้ามีการ `update` อาจทำให้ไฟล์ที่แก้ไขไว้หายไป  ถ้าใครรู้ก็รู้วิธีอื่นๆ แนะนำมาได้นะครับ

ก่อนอื่นเราต้อง ดาวน์โหลดฟอนต์ไทยมาก่อนครับ ([Download](/downloads/thaifont.zip))

- แตกซิบไฟล์แล้วนำฟอนต์ทั้งหมดไปไว้ที่ `vendor/kartik-v/mpdf/ttfont/`
- จากนั้นทำการแก้ไขไฟล์ `vendor/kartik-v/mpdf/config_fonts.php` เพิ่มชื่อฟอนต์เข้าไป
```php
 $this->fontdata = array(
   "thsaraban" => array(
        'R' => "THSarabunNew.ttf",
        'B' => "THSarabunNew-Bold.ttf",
        'I'  => "THSarabunNew-Italic.ttf",
        'BI'   =>  "THSarabunNew-BoldItalic.ttf"
        ),
 ```
- และเพิ่ม `thsaraban` เข้าไปที่
```php
$this->sans_fonts = array('thsaraban',
```
- ไปที่ไฟล์ `vendor/kartik-v/mpdf/config_lang2fonts.php` แล้วค้นหาคำว่า Thai แล้วแก้เป็นดังนี้
```php
  CASE "th":  CASE "tha":	// THAI
  $unifont = "thsaraban";
  break;
```
- แก้ไขไฟล์ `vendor/kartik-v/yii2-mpdf/Pdf.php`  เพิ่ม `thsaraban` เข้าไป
```php
public function setApi()
   {
       $this->_mpdf = new mPDF(
           //$this->mode,
           'thsaraban',
           $this->format,
           $this->defaultFontSize,
           $this->defaultFont,
           $this->marginLeft,
           $this->marginRight,
           $this->marginTop,
           $this->marginBottom,
           $this->marginHeader,
           $this->marginFooter,
           $this->orientation
       );
   }
```
- แก้ไขไฟล์ css ที่ `vendor/kartik-v/yii2-mpdf/assets/kv-mpdf-bootstrap.css` เพิ่มฟอนต์ `thsaraban` เข้าไป
```css
body {
  font-family: "thsaraban","Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: 14px;
  line-height: 1.42857143;
  color: #333;
  background-color: #fff;
}
```

### ตัวอย่างไฟล์
![thaipdf](/images/pdf-thai.png)
