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

จากนั้นเราจะทำการเปลี่ยนจาก input เป็น file โดยใช้ [File Input Widget](http://demos.krajee.com/widgets#fileinput) ทำการเรียกใช้ดังนี้

ทำการเรียก File Input Widget เข้ามาก่อน

```php
<?php
use kartik\widgets\FileInput;
```
จากนั้นเปลี่ยน `textInput` จากของเดิมแบบนี้

```php
<?= $form->field($model, 'covenant')->textInput(['maxlength' => 100]) ?>
```
และทำการเรียกใช้งาน `FileInput` และกำหนดค่าต่างๆ

- `'options' => ['accept' => 'image/*']` ถ้าเราใช้ upload รูปภาพอย่างเดียว
- `initialPreview` เป็นการนำข้อมูลที่เคยบันทึกไปแล้วมาแสดงที่ thumbnail
- `allowedFileExtensions` กำหนดให้เราสามารถอัพโหลดไฟล์ format ใหนบ้าง
- `showRemove` ให้มีปุ่มยกเลิกในกรณีที่เราต้องการเลือกไฟล์ใหม่
- `showUpload` เป็นปุ่ม upload file แบบ ajax ซึ่งตอนนี้เรายังไม่ใช้กับฟิวด์นี้

> สำหรับฟิวด์ `covenang` ผมกำหนดให้สามารถอัพโหลดได้ทีละไฟล์

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

ส่วนฟิวด์ docs ผมจะให้สามารถอัพโหลดได้มากกว่า 1 ไฟล์

```php

<?= $form->field($model, 'docs')->widget(FileInput::classname(), [
   //'options' => ['accept' => 'image/*'],
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

จะได้ file upload แบบนี้
![upload](/images/upload-file.png)
