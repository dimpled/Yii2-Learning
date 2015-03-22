สร้างฟอร์ม Upload File และฟอร์ม Upload Files ด้วย AJAX
---
![upload file](/images/upload-file-ajax.png)

เชื่อว่าหลายๆ คนคงเคยปวดหัวกับการสร้างฟอร์มเพื่อทำการอัพโหลดไฟล์ต่างๆ หรือไฟล์รูปภาพเองก็ตาม ถ้าเป็นฟอร์มอัพโหลดแบบธรรมดาก็คงไม่เท่าไหร่ แต่ถ้าเป็นแบบ ajax ก็คงจะปวดหัวไม่ใช่น้อย.. T_T' เดี่ยววันเราจะมาลองทำฟอร์ม Upload แบบ ajax กันซึ่งพระเอกของเราวันนี้คือ [fileinput](http://demos.krajee.com/widgets#fileinput) เป็น Widget ของ krajee เจ้าเก่า ซึ่งรวมอยู่ในแพ็คเก็จ [Yii2 Widget](http://demos.krajee.com/widgets) อยู่แล้ว ติดตั้งทีเดียวได้ครบเลย ^^

ในตัวอย่างนี้จะสร้างฟอร์มไว้เก็บข้อมูล freelance ข้อมูลไฟล์สัญญา ไฟล์ข้อมูลต่างๆ ไฟล์ข้อมูลรูปภาพ

## การติดตั้ง Widget

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
หลังจากสร้าง Model เสร็จเราจะทำการปรับแต่ง `Rules()` เพื่อเป็นการ `Validating Input` ของเรา ซึ่ง Yii 2 มีให้มาครบเลย เช่น email, date, url ฯลฯ ในฟอร์มนี้เราจะใช้ `file` เพราะเราจะทำการ upload file
> [รายการ validate ทั้งหมดดูได้ที่นี่](http://www.yiiframework.com/doc-2.0/yii-validators-validator.html)

เปิดไฟล์ /models/Freelance.php

เพิ่มฟิวด์ `covenant` ที่จะเก็บข้อมูลไฟล์สัญญาจ้าง และ `docs` เก็บไฟล์เอกสารที่เกี่ยวข้องทั้งหมด กำหนด type validate เป็น `file`
```php
<?php
//.......

public function rules()
{
    return [
        [['title'],'required'],
        [['description'], 'string'],
        [['start_date', 'end_date', 'succes_date', 'create_date'], 'safe'],
        [['ref'], 'string', 'max' => 20],
        [['title'], 'string', 'max' => 255],
        [['covenant'],'file','maxFiles'=>1], //<---
        [['docs'],'file','maxFiles'=>10] //<---
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
จากนั้นก็ปรับ layout from ตามใจชอบ [ดูรายละเอียดการปรับ layout form ได้ที่นี่](/tutorial/create-form.md)

![form-upload-layout.](/images/upload-file/form-upload-layout.png)

```php
<?php

use yii\helpers\Html;
use yii\widgets\ActiveForm;

/* @var $this yii\web\View */
/* @var $model app\models\Freelance */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="freelance-form">

    <?php $form = ActiveForm::begin(); ?>

     <?= $form->field($model, 'ref')->hiddenInput()->label(false); ?>

    <?= $form->field($model, 'title')->textInput(['maxlength' => 255]) ?>

    <?= $form->field($model, 'description')->textarea(['rows' => 3]) ?>

    <?= $form->field($model, 'covenant')->textInput(['maxlength' => 100]) ?>

    <?= $form->field($model, 'docs')->textarea(['rows' => 6]) ?>
    <div class="row">
        <div class="col-sm-4 col-md-4"> <?= $form->field($model, 'start_date')->textInput() ?></div>
        <div class="col-sm-4 col-md-4"><?= $form->field($model, 'end_date')->textInput() ?></div>
        <div class="col-sm-4 col-md-4"><?= $form->field($model, 'succes_date')->textInput() ?></div>
    </div>

    <div class="form-group">
        <?= Html::submitButton('<i class="glyphicon glyphicon-plus"></i> '.($model->isNewRecord ? 'Create' : 'Update'), ['class' => ($model->isNewRecord ? 'btn btn-success' : 'btn btn-primary').' btn-lg btn-block']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>

```

จากนั้นเราจะทำการเปลี่ยนจาก input เป็น file โดยใช้ [File Input Widget](http://demos.krajee.com/widgets#fileinput) ทำการเรียก File Input Widget เข้ามาก่อน

```php
use kartik\widgets\FileInput;
```

จากนั้นเปลี่ยน `textInput` จากของเดิมแบบนี้

```php
<?= $form->field($model, 'covenant')->textInput(['maxlength' => 100]) ?>
```
และทำการเรียกใช้งาน `FileInput` และกำหนดค่าต่างๆ
```php

<?= $form->field($model, 'covenant')->widget(FileInput::classname(), [
    //'options' => ['accept' => 'image/*'],
    'pluginOptions' => [
        'initialPreview'=>[],
        'allowedFileExtensions'=>['pdf'],
        'showPreview' => false,
        'showRemove' => true,
        'showUpload' => false
     ]
]); ?>

```

- `'options' => ['accept' => 'image/*']` ถ้าเราใช้ upload รูปภาพอย่างเดียว
- `initialPreview` เป็นการนำข้อมูลที่เคยบันทึกไปแล้วมาแสดงที่ thumbnail
- `allowedFileExtensions` กำหนดให้เราสามารถอัพโหลดไฟล์ format ใหนบ้าง
- `showRemove` ให้มีปุ่มยกเลิกในกรณีที่เราต้องการเลือกไฟล์ใหม่
- `showUpload` เป็นปุ่ม upload file แบบ ajax ซึ่งตอนนี้เรายังไม่ใช้กับฟิวด์นี้

 [อ่านรายละเอียดเพิ่มเติม](http://plugins.krajee.com/file-input)

> สำหรับฟิวด์ `covenang` ผมกำหนดให้สามารถอัพโหลดได้ทีละไฟล์


ส่วนฟิวด์ `docs` ผมจะให้สามารถอัพโหลดได้มากกว่า 1 ไฟล์ สังเกตุตรงชื่อผมจะใส่ `docs[]` เพื่อให้ส่งไฟล์ไปอัพโหลดได้ทีละหลายๆ ไฟล์ และเซ็ตค่า widget `multiple=true`

> อย่าลืมเช็ค `Server Environment` php.ini กำหนดค่าเท่าไหร่แล้วแต่ความเหมาะสมของเรานะครับ
- file_uploads = On
- max_file_uploads = 10
- upload_max_filesize = 50M
- post_max_size = 50M

```php

<?= $form->field($model, 'docs[]')->widget(FileInput::classname(), [
'options' => [
    //'accept' => 'image/*',
    'multiple' => true
],
'pluginOptions' => [
    'initialPreview'=>[],
    //'allowedFileExtensions'=>['pdf'],
    'showPreview' => false,
    'showCaption' => true,
    'showRemove' => true,
    'showUpload' => false
 ]
]); ?>

```
หลังจากนั้นลองดูฟอร์ที่เราได้ปรับแต่งแล้ว ก็จะได้ฟอร์มอัพโหลดที่ค่อนข้างดูดีทีเดียว ไม่ใช่แค่สวย แต่มันยังทำให้เราใช้งานได้ง่ายขึ้นอีกด้วย แจ่มๆ ^^

![upload](/images/upload-file/add-widget.png)

## ทดลองการ Validate ไฟล์

### ฟิวด์แรก `covenant`

ทดลองเลือกไฟล์ที่ไม่ใช่ pdf ก็จะได้ error แบบนี้ และสามารถเลือกได้แค่ไฟล์เดียวเพราะเรากำหนดไว้ที่ `Rules()` ใน model ไว้แล้ว

![upload](/images/upload-file/single-error.png)

ถ้าเลือกไฟล์ถูกต้อง ก็จะแสดงแบบนี้ เห้ย! สวยใช้ได้เลย

![upload](/images/upload-file/single-success.png)


### ฟิวด์ที่สอง `docs`

 ฟิวด์นี้เรากำหนดไว้ว่าให้สามารถอัพโหลดได้ทีละหลายๆ ไฟล์ และเลือกได้หลายๆ นามสกุลไฟล์ `'pdf','doc','docx','xls','xlsx'`

ลองเลือกหลายๆ ไฟล์
> ต้องกดปุ่ม Ctl หรือ command ถ้าเป็น mac ค้างไว้แล้วเลือกไฟล์

![upload](/images/upload-file/multiple-success.png)

ลองเลือกเกินที่กำหนดไว้

![upload](/images/upload-file/multiple-error-max.png)

ลองเลือกนามสกุลไฟล์ที่ไม่ได้ระบุไว้

![upload](/images/upload-file/multiple-error.png)


> จริงๆ แล้ว UploadFile ของเดิมๆ ที่มากับ yii 2 ก็ใช้ได้นะครับเพียงแค่หน้าตาอาจไม่สวย

 อีกนิดเราสามารถให้ Widget โชว์ thumbnail ไฟล์ที่เราได้เลือกไว้ได้โดยปรับการตั้งค่าใหม่เป็น
```php
'showPreview' => true,
```
จะแสดงรายการที่เราได้เลือกไว้ แบบนี้ ! โอ้วพระสงฆ์

![upload](/images/upload-file/show-thumbnail.png)

เสร็จสิ้นในการเตรียมในส่วนของ front end เดี่ยวไปสร้าง controller กัน


## Create Controller
