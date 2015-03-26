สร้างฟอร์ม Upload File และฟอร์ม Upload Files ด้วย AJAX
---
![upload file](/images/upload-file-ajax.png)

เชื่อว่าหลายๆ คนคงเคยปวดหัวกับการสร้างฟอร์มเพื่อทำการอัพโหลดไฟล์ต่างๆ หรือไฟล์รูปภาพเองก็ตาม ถ้าเป็นฟอร์มอัพโหลดแบบธรรมดาก็คงไม่เท่าไหร่ แต่ถ้าเป็นแบบ ajax ก็คงจะปวดหัวไม่ใช่น้อย.. T_T' เดี่ยววันเราจะมาลองทำฟอร์ม Upload แบบ ajax กันซึ่งพระเอกของเราวันนี้คือ [fileinput](http://demos.krajee.com/widgets#fileinput) เป็น Widget ของ krajee เจ้าเก่า ซึ่งรวมอยู่ในแพ็คเก็จ [Yii2 Widget](http://demos.krajee.com/widgets) อยู่แล้ว ติดตั้งทีเดียวได้ครบเลย ^^

ในตัวอย่างนี้มี 3 แบบ
- Upload ทีละไฟล์เก็บข้อมูลเป็น json ใช้ฟิวด์เดียว
- Upload ทีละหลายๆ ไฟล์ เก็บข้อมูลเป็น json ใช้ไฟล์เดียว
- Upload ทีละ 1 ไฟล์ หรือทีละหลายๆ ไฟล์ก็ได้ โดยใช้ Ajax

> เก็บเป็น json ก็เพราะสามารถเก็บข้อมูลได้เยอะขึ้น และไม่จำเป็นต้องเพิ่มฟิวด์

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
หลังจากนั้นลองดูฟอร์มที่เราได้ปรับแต่งแล้ว ก็จะได้ฟอร์มอัพโหลดที่ค่อนข้างดูดีทีเดียว ไม่ใช่แค่สวย แต่มันยังทำให้เราใช้งานได้ง่ายขึ้นอีกด้วย แจ่มๆ ^^

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

เสร็จสิ้นในการเตรียมในส่วนของ front end เดี่ยวไปสร้าง controller และ model กัน

##  ปรับปรุง Model Freelance
เพื่อให้สามารถเรียก path ที่เก็บไฟล์และ url ที่เก็บไฟล์ได้โดยเราจะสร้างฟังก์ชั่นชึ้นมา 2 ตัวคือ `getUploadPath()`,`getUploadUrl()`

โดยเป็น static  function เพื่อให้สามารถเรียกใช้งานได้ง่ายๆ โดยไม่ต้อง new Object
และประการตัวแปร UPLOAD_FOLDER เป็น static เหมือนกันเพื่อกำหนดค่า folder ที่อัพโหลดได้เวลาเปลี่ยนชื่อจะได้เปลี่ยนที่เดียว

```php

const UPLOAD_FOLDER = 'freelances';

//...........

public static function getUploadPath(){
    return Yii::getAlias('@webroot').'/'.self::UPLOAD_FOLDER.'/';
}

public static function getUploadUrl(){
    return Url::base(true).'/'.self::UPLOAD_FOLDER.'/';
}
```

## Create Controller
เราจะทำการแก้ไข Controller `Freelance` โดยหลักๆ เราจะสร้าง function สำหรับ upload  file ขี้นมา 2 ตัว แบบอัพโหลดทีละไฟล์ และแบบอัพโหลดทีละหลายๆ ไฟล์

### สร้างฟังก์ชั่นอัพโหลดทีละ 1 ไฟล์
เราจะเก็บข้อมูลในฟิวด์ `covenant` เป็น json เพื่อให้สามารถทำการเปลี่ยนชื่อไฟล์เพื่อเก็บบน server โดยไม่มีโอกาสซ้ำชื่อกันและเมื่อดาวน์โหลดไฟล์ก็จะส่งเป็นชื่อไฟล์เดิมลงมา

ฟังก์ชั่นนี้จะรับค่า 2 ค่าคือ $model,$tempFile

 `$tempFile` มีไว้เพื่อเก็บชื่อไฟล์เดิม เพื่อเอาไว้ใช้ในกรณีแก้ไขข้อมูล หากการแก้ไขข้อมูลนั้นไม่ได้มีการอัพโหลดไฟล์ใหม่ ก็จะใช้ค่าเดิมจาก `$tempFile` เพราะเวลาที่ submit form แล้วไม่ได้อัพโหลดไฟล์ใหม ค่าจะเป็น null ทำให้ค่าเดิมที่เก็บไว้ก่อนหน้านี้จะเป็น null ตามไปด้วย เราจึงใช้ค่าจาก $tempFile มาใช้เพื่อให้ข้อมูลเหมือนเดิม

 จากนั้นจะทำการเก็บชื่อไฟล์เดิมและชื่อไฟล์ใหม่ไว้เป็น array และนำมาแปลงค่าให้เป็น json แล้วส่งค่ากลับออกไปเป็น string ในรูปแบบ json เพื่อนำไปบันทึกข้อมูล

use file เพิ่มเติม

```php
use yii\web\UploadedFile;
use yii\helpers\BaseFileHelper;
use yii\helpers\Json;
use yii\helpers\ArrayHelper;
```

สร้าง function

```php
private function uploadSingleFile($model,$tempFile=null){
        $file = [];
        $json = '';
        try {
             $UploadedFile = UploadedFile::getInstance($model,'covenant');
             if($UploadedFile !== null){
                 $oldFileName = $UploadedFile->basename.'.'.$UploadedFile->extension;
                 $newFileName = md5($UploadedFile->basename.time()).'.'.$UploadedFile->extension;
                 $UploadedFile->saveAs(Freelance::UPLOAD_FOLDER.'/'.$model->ref.'/'.$newFileName);
                 $file[$newFileName] = $oldFileName;
                 $json = Json::encode($file);
             }else{
                $json=$tempFile;
             }
        } catch (Exception $e) {
            $json=$tempFile;
        }
        return $json ;
    }
```

### สร้างฟังก์ชั่นอัพโหลดทีละหลายๆ ไฟล์

ฟังก์ชันนี้ก็จะคล้ายๆ กับ `uploadSingleFile()` แต่ต่างกันที่สามารถอัพโหลดได้ทีละหลายๆ ไฟล์
และก็จะรับค่า parameter  เหมือนกันคือ $model,$tempFile

```php

private function uploadMultipleFile($model,$tempFile=null){
            $files = [];
            $json = '';
            $tempFile = Json::decode($tempFile);
            $UploadedFiles = UploadedFile::getInstances($model,'docs');
            if($UploadedFiles!==null){
               foreach ($UploadedFiles as $file) {
                   try {   $oldFileName = $file->basename.'.'.$file->extension;
                           $newFileName = md5($file->basename.time()).'.'.$file->extension;
                           $file->saveAs(Freelance::UPLOAD_FOLDER.'/'.$model->ref.'/'.$newFileName);
                           $files[$newFileName] = $oldFileName ;
                   } catch (Exception $e) {

                   }
               }
               $json = json::encode(ArrayHelper::merge($tempFile,$files));
            }else{
               $json = $tempFile;
            }
           return $json;
   }
```

### สร้างฟังก์ชันเพื่อสร้างโฟลเดอร์สำหรับเก็บไฟล์

ฟังชั่นนี้ก็ง่ายๆ เอาไว้สำหรับสร้าง folder ไว้เก็บไฟล์ในแต่ละ id เพื่อให้แยกไฟล์ได้เป็นระเบียบและเพื่อการค้นหาด้วย

```php
private function CreateDir($folderName){
    if($folderName != NULL){
        $basePath = Freelance::getUploadPath();
        if(BaseFileHelper::createDirectory($basePath.$folderName,0777)){
            BaseFileHelper::createDirectory($basePath.$folderName.'/thumbnail',0777);
        }
    }
    return;
}

```

### เรียกใช้งานตอน Create

เราจะเรียกใช้งานที่ actionCreate เพื่ออัพโหลดไฟล์ โดยของเดิมจะเป็นแบบนี้

```php
public function actionCreate()
{
      $model = new Freelance();

      if ($model->load(Yii::$app->request->post()) && $model->save()) {
              return $this->redirect(['view', 'id' => $model->id]);
      } else {
          return $this->render('create', [
              'model' => $model,
          ]);
      }
}
```
จากนั้นเพิ่มอัพโหลดไฟล์เข้าไป ตามด้านล่าง

```php
public function actionCreate()
{
    $model = new Freelance();

    if ($model->load(Yii::$app->request->post()) ) {

        $this->CreateDir($model->ref);

        $model->covenant = $this->uploadSingleFile($model);
        $model->docs = $this->uploadMultipleFile($model);

        if($model->save()){
             return $this->redirect(['view', 'id' => $model->id]);
        }

    } else {
         $model->ref = substr(Yii::$app->getSecurity()->generateRandomString(),10);
    }

    return $this->render('create', [
        'model' => $model,
    ]);
}
```

หลักการทำงานของฟังก์ชันคือ

ในครั้งแรก
- เช็คว่ามีการ submit form มาหรือไม่
- ถ้าไม่มีก็จะทำการสร้างรหัส random string มาเก็บไว้ที่ฟิวด์ ref เพื่อใช้ในกรณีอัพโหลดแบบ ajax เพื่อให้อัพโหลดไฟล์ได้ก่อนที่จะมีการ submit เพื่อบันทึกข้อมูล หากไม่มีเราจะต้องใช้ pk ซึ่งมันต้องรอบันทึกก่อน เราจึงทำการสร้างรหัสนี้ขึ้นมาใช้
- จากนั้นก็ render form ปกติ

เมื่อมีการ submit
- ทำการสร้าง folder เพื่อเก็บไฟล์โดยใช้ ref
- เมื่อมีการ submit ก็จะมีการเรียกใช้งาน `uploadSingleFile`,`uploadMultipleFile` และเมื่ออัพโหลดไฟล์เสร็จจะคืนค่ากลับมาเป็น string json ถ้าไม่มีไฟล์ก็จะคืนค่าเป็น null


### เรียกใช้งานตอน Update

จะคล้ายกับ create ต่างกันนิดหน่อยตรงที่ต้องเก็บค่าไฟล์เดิมไว้ก่อน เพื่อใช้ในกรณีไม่ได้มีการอัพโหลดไฟล์

```php
public function actionUpdate($id)
{
    $model = $this->findModel($id);
    $tempCovenant = $model->covenant;
    $tempDocs     = $model->docs;
    if ($model->load(Yii::$app->request->post())) {
        $model->covenant = $this->uploadSingleFile($model,$tempCovenant);
        $model->docs = $this->uploadMultipleFile($model,$tempDocs);
        if($model->save()){
            return $this->redirect(['view', 'id' => $model->id]);
        }
    }

    return $this->render('update', [
        'model' => $model,
    ]);
}
```

### ทดลองอัพโหลดไฟล์

เมื่ออัพโหลดไฟล์เสร็จ เราจะได้ข้อมูล json ออกมาแบบนี้

![view-upload](/images/upload-file/view-upload.png)

โดยข้อมูลจะเก็บเป็น key กับ value ซึ่ง key จะเป็นชื่อไฟล์จริงๆ ใน server และ value จะเป็นชื่อไฟล์ก่อนอัพโหลดไฟล์

เราจะทำการเปลี่ยนข้อมูลจาก json เป็น link เพื่อให้สามารถคลิก download ได้โดยเพิ่ม function `listDownloadFiles()` ใน Model Freelance ฟังก์ชันนี้จะใช้ได้ทั้ง 2 ฟิวด์ docs,covenant

```php
public function listDownloadFiles($type){
 $docs_file = '';
 if(in_array($type, ['docs','covenant'])){

         $data = $type==='docs'?$this->docs:$this->covenant;
         $files = Json::decode($data);
        if(is_array($files)){
             $docs_file ='<ul>';
             foreach ($files as $key => $value) {
                $docs_file .= '<li>'.Html::a($value,['/freelance/download','id'=>$this->id,'file'=>$key,'file_name'=>$value]).'</li>';
             }
             $docs_file .='</ul>';
        }
 }

 return $docs_file;
}
```
จะแสดงผลแบบนี้

![view-upload](/images/upload-file/view-upload2.png)


### สร้าง function สำหรับ download file
ในการแสดงผลในหน้า view นั้นจะยังคลิก อัพโหลดไม่ได้เราต้องสร้าง function สำหรับ download ไฟล์ เพื่อไม่ให้เป็นการ link ไฟล์โดยตรง

เพิ่ม actionDownload ที่ controllers/Freelance.php

```php
public function actionDownload($id,$file,$file_name){
    $model = $this->findModel($id);
     if(!empty($model->ref) && !empty($model->covenant)){
            Yii::$app->response->sendFile($model->getUploadPath().'/'.$model->ref.'/'.$file,$file_name);
    }else{
        $this->redirect(['/freelance/view','id'=>$id]);
    }
}
```
เราจะได้ไฟล์ตามชื่อไฟล์จริง ที่เคยอัพโหลดไว้ และ actionDownload จะส่งไฟล์ให้ client โดยไม่ได้ link ไฟล์ตรงๆ

![view-upload](/images/upload-file/view-upload3.png)
