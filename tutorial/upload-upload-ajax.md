สร้างฟอร์ม Upload File และฟอร์ม Upload Files ด้วย AJAX
---
![upload file](/images/upload-file-ajax.png)

เชื่อว่าหลายๆ คนคงเคยปวดหัวกับการสร้างฟอร์มเพื่อทำการอัพโหลดไฟล์ต่างๆ หรือไฟล์รูปภาพเองก็ตาม ถ้าเป็นฟอร์มอัพโหลดแบบธรรมดาก็คงไม่เท่าไหร่ แต่ถ้าเป็นแบบ ajax ก็คงจะปวดหัวไม่ใช่น้อย.. T_T' เดี่ยววันเราจะมาลองทำฟอร์ม Upload แบบ ajax กันซึ่งพระเอกของเราวันนี้คือ [fileinput](http://demos.krajee.com/widgets#fileinput) เป็น Widget ของ krajee เจ้าเก่า ซึ่งรวมอยู่ในแพ็คเก็จ [Yii2 Widget](http://demos.krajee.com/widgets) อยู่แล้ว ติดตั้งทีเดียวได้ครบเลย ^^

ในตัวอย่างนี้จะสร้างฟอร์มไว้เก็บข้อมูล freelance ข้อมูลไฟล์สัญญา ไฟล์ข้อมูลต่างๆ ไฟล์ข้อมูลรูปภาพ

## การติดตั้ง Widget
ในตัวอย่างนี้เราใช้ Widget [fileinput](http://demos.krajee.com/widgets#fileinput) ซึ่งจะเป็นเพ็คเก็จที่รวมอยุ่ใน [Yii2 Widgets](http://demos.krajee.com/widgets) ซึ่งจะมี widget อื่นๆ อีกหลายตัว


รันคำสั่งเพื่อทำการติดตั้ง
```
composer require kartik-v/yii2-widgets "*"
```
หรือเพิ่ม
```
"kartik-v/yii2-widgets": "*"
```
ที่แท็ค require ในไฟล์ composer.json จากนั้น `composer update`

## Create Model
ทำการสร้าง gii Model `Freelance`, และทำการสร้าง gii CRUD `Freelance`

### Validating Input
หลังจากสร้าง Model เสร็จเราจะทำกางปรับแต่ง `Rules()` เพื่อเป็นการ `Validating Input` ของเรา ซึ่ง Yii 2 มีให้มาครบเลย เช่น email, date, url ฯลฯ ในฟอร์มนี้เราจะใช้ `file` เพราะเราจะทำการ upload file
> [รายการ validate ทั้งหมดดูได้ที่นี่](http://www.yiiframework.com/doc-2.0/yii-validators-validator.html)

เปิดไฟล์ /models/Freelance.php

เพิ่มฟิวด์ `covenant` ที่จะเก็บข้อมูลไฟล์สัญญาจ้าง และ `docs` เก็บไฟล์เอกสารที่เกี่ยวข้องทั้งหมด กำหนด type validate เป็น `file`
```php
<?php
//.......

public function rules()
{
    return [
        [['title','covenant'],'required'],
        [['description', 'docs'], 'string'],
        [['start_date', 'end_date', 'succes_date', 'create_date','docs'], 'safe'],
        [['ref'], 'string', 'max' => 20],
        [['title'], 'string', 'max' => 255],
        [['covenant','docs'],'file'] //<-----
    ];
}

//.......

```

## Create Form
ในส่วนของ form เราต้องเพิ่ม attribute `enctype="multipart/form-data"` ส่วนใหญ่หลายๆ คนอาจลืมผมก็เคยลืม ฮา..

ไปที่ไฟล์ /views/freelance/_form.php แก้ไข `ActiveForm` ให้เป็นดังนี้

```php
<?php $form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]) ?>
```
จากนั้นก็ปรับ layout from ตามใจชอบ [ดูรายละเอียดการปรับ layout form ได้ที่นี่](/tutorial/create-from.md)

```php
<?=  $form->field($model, 'resume')->widget(FileInput::classname(), [
'options' => ['accept' => 'image/*'],
'pluginOptions' => [
    'initialPreview'=>[],
    'allowedFileExtensions'=>['pdf'],
    'showPreview' => false,
    'showCaption' => true,
    'showRemove' => true,
    'showUpload' => false
 ]
]); ?>
```
จะได้ file upload แบบนี้
![upload](/images/upload-file.png)
