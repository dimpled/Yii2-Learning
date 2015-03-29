สร้างฟอร์ม Upload Files ด้วย AJAX
---
![upload file](/images/upload-file-ajax.png)

เชื่อว่าหลายๆ คนคงเคยปวดหัวกับการสร้างฟอร์มเพื่อทำการอัพโหลดไฟล์ต่างๆ หรือไฟล์รูปภาพเองก็ตาม ถ้าเป็นฟอร์มอัพโหลดแบบธรรมดาก็คงไม่เท่าไหร่ แต่ถ้าเป็นแบบ ajax ก็คงจะปวดหัวไม่ใช่น้อย.. T_T' เดี่ยววันเราจะมาลองทำฟอร์ม Upload แบบ ajax กันซึ่งพระเอกของเราวันนี้คือ [fileinput](http://demos.krajee.com/widgets#fileinput) เป็น Widget ของ krajee เจ้าเก่า ซึ่งรวมอยู่ในแพ็คเก็จ [Yii2 Widget](http://demos.krajee.com/widgets) อยู่แล้ว ติดตั้งทีเดียวได้ครบเลย ^^

เราจะเพิ่มการอัพโหลดแบบ ajax คุณสมบัติหลักคือ
- สามารถอัพโหลดได้ทันทีผ่าน  ajax  โดยไม่ต้องรอ create record
- สามารถกดปุ่ม upload ajax ได้ หรือ กด submit ผ่านฟอร์มได้เช่นกัน
- หากเป็นรูปภาพจะสร้าง thumbnail และ resize อัตโนมัติ
- สามารถกดปุ่มลบรายการที่อัพโหลดได้
- เมื่อมีการลบ record จะตามไปลบข้อมูลทั้ง folder ด้วยและลบรายการทั้งหมดที่ตาราง uploads


## ติดตั้ง extension เพิ่มเติม
เพิ่ม รายชื่อ extension เข้าไปที่ไฟล์ composer.json เพิ่มไว้ภายใต้แท็ก require จากนั้น cd เข้าไปที่ root project ของเราแล้วรันคำสั่ง `composer update`

```json
"kartik-v/yii2-widgets": "*",
"yurkinx/yii2-image": "dev-master",
"2amigos/yii2-date-picker-widget" : "~1.0",
"2amigos/yii2-gallery-widget" : "*"
```

## Gii Model และ Gii CRUD PhotoLibrary
ในตัวอย่างนี้เราจะสร้างระบบเก็บรูปภาพโดยใช้ตารางชื่อว่า photolibrary และสร้างตารางตามนี้

```sql
CREATE TABLE `photo_library` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `ref` varchar(50) DEFAULT NULL COMMENT 'เลข fk กับ upload ใช้กับ upload ajax',
  `event_name` varchar(255) DEFAULT NULL COMMENT 'ชื่องาน',
  `detail` text COMMENT 'รายละเอียด',
  `start_date` datetime DEFAULT NULL COMMENT 'วันที่เริ่มถ่ายภาพ',
  `end_date` datetime DEFAULT NULL COMMENT 'วันที่ถ่ายภาพเสร็จ',
  `location` varchar(255) DEFAULT NULL COMMENT 'สถานที่',
  `customer_name` varchar(150) DEFAULT NULL COMMENT 'ชื่อลูกค้า',
  `customer_mobile_phone` varchar(20) DEFAULT NULL COMMENT 'เบอร์โทร',
  `province_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ref` (`ref`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
```

หลังจากสร้างตารางเรียบร้อย ให้ทำการ gii Model PhotoLibrary และ crud PhotoLibrary

## สร้างฟอร์ม
หลังจากที่ได้ไฟล์จากการ gii crud แล้วเราจะทำการปรับปรุงฟอร์มเพิ่มเติมเพื่อให้สามารถ upload file ได้

เปิดไฟล์ `views\photo-library\_form.php` เพิ่มแท็ก  `enctype` เข้าไป

```php
<?php $form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]); ?>
```

แก้ไขฟิวด์ ref ให้เป็น hidenInput และเซ็ตค่า label เป็น false
```php
<?= $form->field($model, 'ref')->hiddenInput(['maxlength' => 50])->label(false); ?>
```
ทำการเรียกใช้งาน Select2 widget เพื่อแสดงรายชื่อจังหวัดจาก model `Province`

>โครงสร้างตาราง province [ดูได้ที่นี่](https://github.com/dimpled/Yii2-Learning-Source/blob/master/yii2-learning-source.sql)

เรียกใช้งาน
```php
<?= $form->field($model, 'province_id')->widget(Select2::classname(), [
    'data' => ArrayHelper::map(province::find()->all(),'PROVINCE_ID','PROVINCE_NAME'),
    'options' => ['placeholder' => 'เลือกจังหวัด ...'],
    'pluginOptions' => [
        'allowClear' => true
    ],
]);
?>
```

เรียกใช้งาน DatePicker ของ kartik
```php
<div class="row">
         <div class="col-sm-6 col-md-6">
          <?= $form->field($model, 'start_date')->widget(
         DatePicker::className(), [
             'language' => 'th',
              'options' => ['placeholder' => 'Select issue date ...'],
             'pluginOptions' => [
                 'format' => 'yyyy-mm-dd',
                 'todayHighlight' => true
             ]
     ]);?>
         </div>
         <div class="col-sm-6 col-md-6">
          <?= $form->field($model, 'end_date')->widget(
         DatePicker::className(), [
             'language' => 'th',
              'options' => ['placeholder' => 'Select issue date ...'],
             'pluginOptions' => [
                 'format' => 'yyyy-mm-dd',
                 'todayHighlight' => true
             ]
     ]);?>
         </div>
 </div>
```

เรียกใช้งาน FileUpload เพื่อสร้างฟอร์ม upload ajax โดยตัวอย่างนี้จะใช้เฉพาะกับรูปภาพ ซึ่งจริงๆ แล้วเราสามารถกำหนดให้อัพโหลดเป็นไฟล์อะไรก็ได้
`FileInput::widget()` มี property ที่จำเป็นดังนี้
- `overwriteInitial`  กำหนดให้สามารถเพิ่มไฟล์อัพโหลดใหม่เข้าไปได้โดยที่ไม่ลบรายการเก่าออกไป
- `initialPreviewShowDelete` ถ้าโหลดไฟล์มาแสดงให้ทำการแสดงปุ่มลบด้วย
- `initialPreview` รายการรูปภาพที่เราจะมาแสดงใน thumbnail  ซึ่งรับค่าเป็น array html tag img
- `initialPreviewConfig` ค่าคอนฟิกสำหรับตอนแสดง thumbnail เช่น url ที่ลบไฟล์
- `uploadUrl`  url ที่เราจะส่งไฟล์ไปอัพโหลดแบบ ajax
- `uploadExtraData` เราสามารถส่งค่าตัวแปรแนบไปกับการอัพโหลดได้

```php
<div class="form-group field-upload_files">
      <label class="control-label" for="upload_files[]"> ภาพถ่าย </label>
    <div>
    <?= FileInput::widget([
                   'name' => 'upload_ajax[]',
                   'options' => ['multiple' => true,'accept' => 'image/*'], //'accept' => 'image/*' หากต้องเฉพาะ image
                    'pluginOptions' => [
                        'overwriteInitial'=>false,
                        'initialPreviewShowDelete'=>true,
                        'initialPreview'=> $initialPreview,
                        'initialPreviewConfig'=> $initialPreviewConfig,
                        'uploadUrl' => Url::to(['/photo-library/upload-ajax']),
                        'uploadExtraData' => [
                            'ref' => $model->ref,
                        ],
                        'maxFileCount' => 100
                    ]
                ]);
    ?>
    </div>
    </div>

```

ดูโค้ดที่ปรับแต่งทั้งหมด

```php
<?php

use yii\helpers\Html;
use yii\helpers\Url;
use yii\widgets\ActiveForm;
use yii\helpers\ArrayHelper;

use app\models\Province;
use kartik\widgets\Select2;
use kartik\widgets\FileInput;
use kartik\date\DatePicker;
/* @var $this yii\web\View */
/* @var $model app\models\PhotoLibrary */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="photo-library-form">

<?php $form = ActiveForm::begin(['options' => ['enctype' => 'multipart/form-data']]); ?>

    <?=$form->errorSummary($model) ?>

    <?= $form->field($model, 'ref')->hiddenInput(['maxlength' => 50])->label(false); ?>

    <?= $form->field($model, 'event_name')->textInput(['maxlength' => 255]) ?>

    <?= $form->field($model, 'detail')->textarea(['rows' => 3]) ?>

     <div class="row">
            <div class="col-sm-6 col-md-6">
             <?= $form->field($model, 'start_date')->widget(
            DatePicker::className(), [
                'language' => 'th',
                 'options' => ['placeholder' => 'Select issue date ...'],
                'pluginOptions' => [
                    'format' => 'yyyy-mm-dd',
                    'todayHighlight' => true
                ]
        ]);?>
            </div>
            <div class="col-sm-6 col-md-6">
             <?= $form->field($model, 'end_date')->widget(
            DatePicker::className(), [
                'language' => 'th',
                 'options' => ['placeholder' => 'Select issue date ...'],
                'pluginOptions' => [
                    'format' => 'yyyy-mm-dd',
                    'todayHighlight' => true
                ]
        ]);?>
            </div>
    </div>

    <?= $form->field($model, 'location')->textInput(['maxlength' => 255]) ?>

    <?= $form->field($model, 'province_id')->widget(Select2::classname(), [
        'data' => ArrayHelper::map(province::find()->all(),'PROVINCE_ID','PROVINCE_NAME'),
        'options' => ['placeholder' => 'เลือกจังหวัด ...'],
        'pluginOptions' => [
            'allowClear' => true
        ],
    ]);
    ?>
    <div class="row">
        <div class="col-sm-6 col-md-6">
            <?= $form->field($model, 'customer_name')->textInput(['maxlength' => 150]) ?>
        </div>
        <div class="col-sm-6 col-md-6">
            <?= $form->field($model, 'customer_mobile_phone')->textInput(['maxlength' => 20]) ?>
        </div>
    </div>

   <div class="form-group field-upload_files">
      <label class="control-label" for="upload_files[]"> ภาพถ่าย </label>
    <div>
    <?= FileInput::widget([
                   'name' => 'upload_ajax[]',
                   'options' => ['multiple' => true,'accept' => 'image/*'], //'accept' => 'image/*' หากต้องเฉพาะ image
                    'pluginOptions' => [
                        'overwriteInitial'=>false,
                        'initialPreviewShowDelete'=>true,
                       'initialPreview'=> $initialPreview,
                        'initialPreviewConfig'=> $initialPreviewConfig,
                        'uploadUrl' => Url::to(['/photo-library/upload-ajax']),
                        'uploadExtraData' => [
                            'ref' => $model->ref,
                        ],
                        'maxFileCount' => 100
                    ]
                ]);
    ?>
    </div>
    </div>

    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? 'Create' : 'Update', ['class' => ($model->isNewRecord ? 'btn btn-success' : 'btn btn-primary').' btn-lg btn-block']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>

```

แก้ไขไฟล์ `views/photo-library/create.php`

เราจะทำการส่งค่า `initialPreview` และ `initialPreviewConfig` เป็น array ว่างๆ ไปเพราะในตอน create ยังไม่มีข้อมูลที่จะแสดง เราจึงจำเป็นต้องส่ง array ว่างๆ ไปก่อน
```php
<?php

use yii\helpers\Html;


/* @var $this yii\web\View */
/* @var $model app\models\PhotoLibrary */

$this->title = 'Create Photo Library';
$this->params['breadcrumbs'][] = ['label' => 'Photo Libraries', 'url' => ['index']];
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="photo-library-create">

    <h1><?= Html::encode($this->title) ?></h1>

    <?= $this->render('_form', [
        'model' => $model,
        'initialPreview'=>[],
        'initialPreviewConfig'=>[]
    ]) ?>

</div>

```

แก้ไขไฟล์ `views/photo-library/update.php` ใน actionUpdate เราจะทำการส่งค่า `initialPreview` และ `initialPreviewConfig`  ไปเพื่อให้ widget แสดง thumbnail รูปภาพที่เคยอัพโหลดไปแล้ว แต่ในตอนนี้เรายังไม่ได้สร้างฟังชันเราจะส่ง array ว่างๆ  ไปก่อน

```php
<?php

use yii\helpers\Html;

/* @var $this yii\web\View */
/* @var $model app\models\PhotoLibrary */

$this->title = 'Update Photo Library: ' . ' ' . $model->id;
$this->params['breadcrumbs'][] = ['label' => 'Photo Libraries', 'url' => ['index']];
$this->params['breadcrumbs'][] = ['label' => $model->id, 'url' => ['view', 'id' => $model->id]];
$this->params['breadcrumbs'][] = 'Update';
?>
<div class="photo-library-update">

    <h1><?= Html::encode($this->title) ?></h1>

    <?= $this->render('_form', [
        'model' => $model,
        'initialPreview'=>[],
        'initialPreviewConfig'=>[]
    ]) ?>

</div>

```

ลองทดสอบรันจะได้ฟอร์มตามตัวอย่างนี้
![](/images/upload-file/upload-ajax-form.png)

## เพิ่ม function ใน Model PhotoLibrary

- UPLOAD_FOLDER เป็นตัวแปร static ให้เรากำหนดชื่อโฟลเดอร์ที่ต้องการอัพโหลดได้
- getUploadPath()  เป็นฟังก์ชันไว้เรียกพาทไฟล์จริงใน server
- getUploadUrl()  เป็นฟังก์ชันไว้เรียก url จริงของไฟล์
- getThumbnails()  เป็นฟังก์ชันส่งข้อมูล array ให้ widget Gallry เพื่อแสดง thumbnail
- getProvince() เป็นการสร้าง relation ไปยังตาราง province ด้วยฟิวด์ province_id เพื่อดึงชื่อจังหวัดมาแสดง

 ให้ทำการเพิ่ม function  เข้าไปที่ไฟล์ `models\PhotoLibrary.php` ดังนี้

```php
const UPLOAD_FOLDER='photolibrarys';

// ..........

public static function getUploadPath(){
    return Yii::getAlias('@webroot').'/'.self::UPLOAD_FOLDER.'/';
}

public static function getUploadUrl(){
    return Url::base(true).'/'.self::UPLOAD_FOLDER.'/';
}

public function getThumbnails($ref,$event_name){
     $uploadFiles   = Uploads::find()->where(['ref'=>$ref])->all();
     $preview = [];
    foreach ($uploadFiles as $file) {
        $preview[] = [
            'url'=>self::getUploadUrl(true).$ref.'/'.$file->real_filename,
            'src'=>self::getUploadUrl(true).$ref.'/thumbnail/'.$file->real_filename,
            'options' => ['title' => $event_name]
        ];
    }
    return $preview;
}

public function getProvince()
{
  return $this->hasOne(Province::className(),['PROVINCE_ID'=>'province_id']);
}
```

## สร้าง Function ใน Controller PhotoLibrary สำหรับการอัพโหลด Ajax




`Uploads()` ฟังก์ชันนี้มีหน้าทีรับไฟล์จาก ajax from แล้วส่งข้อมูลมาทำการอัพโหลดไฟล์ สามารถอัพได้ทั้งภาพและไฟล์ ถ้าเป็นภาพจะทำการ resize และสร้าง thumbnail ให้

```php
private function Uploads($isAjax=false) {
             if (Yii::$app->request->isPost) {
                $images = UploadedFile::getInstancesByName('upload_ajax');
                if ($images) {

                    if($isAjax===true){
                        $ref =Yii::$app->request->post('ref');
                    }else{
                        $Freelance = Yii::$app->request->post('Freelance');
                        $ref = $Freelance['ref'];
                    }

                    $this->CreateDir($ref);

                    foreach ($images as $file){
                        $fileName       = $file->baseName . '.' . $file->extension;
                        $realFileName   = md5($file->baseName.time()) . '.' . $file->extension;
                        $savePath       = Freelance::UPLOAD_FOLDER.'/'.$ref.'/'. $realFileName;
                        if($file->saveAs($savePath)){

                            if($this->isImage(Url::base(true).'/'.$savePath)){
                                 $this->createThumbnail($ref,$realFileName);
                            }

                            $model                  = new Uploads;
                            $model->ref             = $ref;
                            $model->file_name       = $fileName;
                            $model->real_filename   = $realFileName;
                            $model->save();

                            if($isAjax===true){
                                echo json_encode(['success' => 'true']);
                            }

                        }else{
                            if($isAjax===true){
                                echo json_encode(['success'=>'false','eror'=>$file->error]);
                            }
                        }

                    }
                }
            }
    }

  ```

  `getInitialPreview()` ฟังก์ชันนี้เอาไว้ดึงรายการข้อมูลของตาราง `Uploads` เพื่อแสดงที่ thumbnail ตรงอัพโหลดในตอนที่มีการแก้ไขข้อมูล

  ```php
  private function getInitialPreview($ref) {
        $datas = Uploads::find()->where(['ref'=>$ref])->all();
        $initialPreview = [];
        $initialPreviewConfig = [];
        foreach ($datas as $key => $value) {
            array_push($initialPreview, $this->getTemplatePreview($value));
            array_push($initialPreviewConfig, [
                'caption'=> $value->file_name,
                'width'  => '120px',
                'url'    => Url::to(['/freelance/deletefile-ajax']),
                'key'    => $value->upload_id
            ]);
        }
        return  [$initialPreview,$initialPreviewConfig];
}
```
`isImage()` ฟังก์ชันนี้เอาไว้แยกแยะระหว่างไฟล์และรูปภาพ
```php
public function isImage($filePath){
        return @is_array(getimagesize($filePath)) ? true : false;
}
```
`getTemplatePreview()` เอาไว้เลือก template ที่จะแสดงผลว่าเป็น image หรือไฟล์อื่นๆ
```php
private function getTemplatePreview(Uploads $model){
        $filePath = Freelance::getUploadUrl().$model->ref.'/thumbnail/'.$model->real_filename;
        $isImage  = $this->isImage($filePath);
        if($isImage){
            $file = Html::img($filePath,['class'=>'file-preview-image', 'alt'=>$model->file_name, 'title'=>$model->file_name]);
        }else{
            $file =  "<div class='file-preview-other'> " .
                     "<h2><i class='glyphicon glyphicon-file'></i></h2>" .
                     "</div>";
        }
        return $file;
}
```
`createThumbnail()` เป็น function ที่เอาไว้สร้าง thumbnail เล็กๆ เพื่อให้สามารถแสดงผลได้เร็ว
```php
private function createThumbnail($folderName,$fileName,$width=250){
  $uploadPath   = Freelance::getUploadPath().'/'.$folderName.'/';
  $file         = $uploadPath.$fileName;
  $image        = Yii::$app->image->load($file);
  $image->resize($width);
  $image->save($uploadPath.'thumbnail/'.$fileName);
  return;
}
```
`actionDeletefileAjax()` เป็น action ที่เอาไว้ลบไฟล์ในกรณีที่คลิกลบไฟล์ จาก thumbnail ในตัวอัพโหลด ajax
```php
public function actionDeletefileAjax(){

    $model = Uploads::findOne(Yii::$app->request->post('key'));
    if($model!==NULL){
        $filename  = Freelance::getUploadPath().$model->ref.'/'.$model->real_filename;
        $thumbnail = Freelance::getUploadPath().$model->ref.'/thumbnail/'.$model->real_filename;
        if($model->delete()){
            @unlink($filename);
            @unlink($thumbnail);
            echo json_encode(['success'=>true]);
        }else{
            echo json_encode(['success'=>false]);
        }
    }else{
      echo json_encode(['success'=>false]);  
    }
}
```
เป็นฟังก์ชันลบ folder ที่ใช้เก็บไฟล์ เราจะใช้ในกรณีหากมีการลบ record ก็ให้ตามไปลบไฟล์ทั้งหมดด้วย
```php
private function removeUploadDir($dir){
    BaseFileHelper::removeDirectory(Freelance::getUploadPath().$dir);
}
```

สุดท้ายทำการปรับปรุง actionDelete() ซักเล็กน้อย โดยหากมีการลบ record ก็ให้ทำการลบโฟล์เดอร์ที่เก็บไฟล์นั้นด้วย โดยใช้ `removeUploadDir()` ที่เราสร้างไว้แล้วก่อนหน้านี้

```php
public function actionDelete($id)
{
    $model = $this->findModel($id);
    //remove upload file & data
    $this->removeUploadDir($model->ref);
    Uploads::deleteAll(['ref'=>$model->ref]);

    $model->delete();

    return $this->redirect(['index']);
}
```

## สร้าง actionUploadAjax
เป็น function ที่เอาไว้รองรับการอัพโหลดไฟล์ผ่าน ajax ซึ่งเราจะใช้ Upload() ที่เราสร้างไว้ โดยรับค่า parameter 1 ตัวคือ $isAjax เพื่อไปเช็คว่าเป็นการอัพโหลดผ่าน ajax หรือผ่านการ submission form จากนั้นให้ทดลองทำการอัพโหลดไฟล์ลองดู
```php
public function actionUploadAjax(){
       $this->Uploads(true);
}
```

## เรียกใช้งานที่ actionCreate และ actionUpdate
ปรับปรุงให้ action เดิมๆ สามารถรองรับการอัพโหลดจากฟอร์ม ajax ได้ เพราะหากเค้าไม่คลิกปุ่มอัพโหลด ajax แล้วข้อมูลจะยังไม่ถูกอัพโหลด เราจะแก้ไขโดยการเรียกใช้งาน function Upload() ที่เราได้สร้างไว้และใช้กับ actionUploadAjax() ซึ่งสามารถนำมาใช้กับ acionCreate และ  actionUpdate ได้

โดยเพิ่ม `this->Uploads(false);`เข้าไป
```php
public function actionCreate()
{
    $model = new PhotoLibrary();

    if ($model->load(Yii::$app->request->post()) ) {

        $this->Uploads(false);

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

ส่วน actionUpdate() เพิ่ม getInitialPreview() เข้ามาด้วยเพื่อใช้แสดง thumbnail ในตัว ajax upload
```php
public function actionUpdate($id)
 {
     $model = $this->findModel($id);

     list($initialPreview,$initialPreviewConfig) = $this->getInitialPreview($model->ref);

     if ($model->load(Yii::$app->request->post())) {
         $this->Uploads(false);

         if($model->save()){
              return $this->redirect(['view', 'id' => $model->id]);
         }
     }

     return $this->render('update', [
         'model' => $model,
          'initialPreview'=>$initialPreview,
          'initialPreviewConfig'=>$initialPreviewConfig
     ]);

 }
```
และแก้ไขไฟล์เพิ่มเติม views/freelance/create.php
```php
<?= $this->render('_form', [
       'model' => $model,
       'initialPreview'=> [],
       'initialPreviewConfig'=> [],
   ]) ?>
```
และแก้ไขไฟล์ views/freelance/update.php
```php
<?= $this->render('_form', [
    'model' => $model,
    'initialPreview'=>$initialPreview,
    'initialPreviewConfig'=>$initialPreviewConfig
]) ?>
```

โค้ดทั้งหมด ใน PhotolibraryController()

```php
<?php

namespace app\Controllers;

use Yii;
use app\models\Uploads;
use app\models\PhotoLibrary;
use app\models\PhotoLibrarySearch;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
use yii\filters\VerbFilter;

use yii\helpers\Url;
use yii\helpers\html;
use yii\web\UploadedFile;
use yii\helpers\BaseFileHelper;
use yii\helpers\Json;
use yii\helpers\ArrayHelper;

/**
 * PhotoLibraryController implements the CRUD actions for PhotoLibrary model.
 */
class PhotoLibraryController extends Controller
{
    public function behaviors()
    {
        return [
            'verbs' => [
                'class' => VerbFilter::className(),
                'actions' => [
                    'delete' => ['post'],
                ],
            ],
        ];
    }

    /**
     * Lists all PhotoLibrary models.
     * @return mixed
     */
    public function actionIndex()
    {
        $searchModel = new PhotoLibrarySearch();
        $dataProvider = $searchModel->search(Yii::$app->request->queryParams);

        return $this->render('index', [
            'searchModel' => $searchModel,
            'dataProvider' => $dataProvider,
        ]);
    }

    /**
     * Displays a single PhotoLibrary model.
     * @param integer $id
     * @return mixed
     */
    public function actionView($id)
    {
        return $this->render('view', [
            'model' => $this->findModel($id),
        ]);
    }

    /**
     * Creates a new PhotoLibrary model.
     * If creation is successful, the browser will be redirected to the 'view' page.
     * @return mixed
     */
    public function actionCreate()
    {
        $model = new PhotoLibrary();

        if ($model->load(Yii::$app->request->post()) ) {

            $this->Uploads(false);

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

    /**
     * Updates an existing PhotoLibrary model.
     * If update is successful, the browser will be redirected to the 'view' page.
     * @param integer $id
     * @return mixed
     */
    public function actionUpdate($id)
    {
        $model = $this->findModel($id);

        list($initialPreview,$initialPreviewConfig) = $this->getInitialPreview($model->ref);

        if ($model->load(Yii::$app->request->post())) {
            $this->Uploads(false);

            if($model->save()){
                 return $this->redirect(['view', 'id' => $model->id]);
            }
        }

        return $this->render('update', [
            'model' => $model,
             'initialPreview'=>$initialPreview,
             'initialPreviewConfig'=>$initialPreviewConfig
        ]);

    }

    /**
     * Deletes an existing PhotoLibrary model.
     * If deletion is successful, the browser will be redirected to the 'index' page.
     * @param integer $id
     * @return mixed
     */
    public function actionDelete($id)
    {
        $model = $this->findModel($id);
        //remove upload file & data
        $this->removeUploadDir($model->ref);
        Uploads::deleteAll(['ref'=>$model->ref]);

        $model->delete();

        return $this->redirect(['index']);
    }

    /**
     * Finds the PhotoLibrary model based on its primary key value.
     * If the model is not found, a 404 HTTP exception will be thrown.
     * @param integer $id
     * @return PhotoLibrary the loaded model
     * @throws NotFoundHttpException if the model cannot be found
     */
    protected function findModel($id)
    {
        if (($model = PhotoLibrary::findOne($id)) !== null) {
            return $model;
        } else {
            throw new NotFoundHttpException('The requested page does not exist.');
        }
    }

/*|*********************************************************************************|
  |================================ Upload Ajax ====================================|
  |*********************************************************************************|*/

    public function actionUploadAjax(){
           $this->Uploads(true);
     }

    private function CreateDir($folderName){
        if($folderName != NULL){
            $basePath = PhotoLibrary::getUploadPath();
            if(BaseFileHelper::createDirectory($basePath.$folderName,0777)){
                BaseFileHelper::createDirectory($basePath.$folderName.'/thumbnail',0777);
            }
        }
        return;
    }

    private function removeUploadDir($dir){
        BaseFileHelper::removeDirectory(PhotoLibrary::getUploadPath().$dir);
    }

    private function Uploads($isAjax=false) {
             if (Yii::$app->request->isPost) {
                $images = UploadedFile::getInstancesByName('upload_ajax');
                if ($images) {

                    if($isAjax===true){
                        $ref =Yii::$app->request->post('ref');
                    }else{
                        $PhotoLibrary = Yii::$app->request->post('PhotoLibrary');
                        $ref = $PhotoLibrary['ref'];
                    }

                    $this->CreateDir($ref);

                    foreach ($images as $file){
                        $fileName       = $file->baseName . '.' . $file->extension;
                        $realFileName   = md5($file->baseName.time()) . '.' . $file->extension;
                        $savePath       = PhotoLibrary::UPLOAD_FOLDER.'/'.$ref.'/'. $realFileName;
                        if($file->saveAs($savePath)){

                            if($this->isImage(Url::base(true).'/'.$savePath)){
                                 $this->createThumbnail($ref,$realFileName);
                            }

                            $model                  = new Uploads;
                            $model->ref             = $ref;
                            $model->file_name       = $fileName;
                            $model->real_filename   = $realFileName;
                            $model->save();

                            if($isAjax===true){
                                echo json_encode(['success' => 'true']);
                            }

                        }else{
                            if($isAjax===true){
                                echo json_encode(['success'=>'false','eror'=>$file->error]);
                            }
                        }

                    }
                }
            }
    }

    private function getInitialPreview($ref) {
            $datas = Uploads::find()->where(['ref'=>$ref])->all();
            $initialPreview = [];
            $initialPreviewConfig = [];
            foreach ($datas as $key => $value) {
                array_push($initialPreview, $this->getTemplatePreview($value));
                array_push($initialPreviewConfig, [
                    'caption'=> $value->file_name,
                    'width'  => '120px',
                    'url'    => Url::to(['/photo-library/deletefile-ajax']),
                    'key'    => $value->upload_id
                ]);
            }
            return  [$initialPreview,$initialPreviewConfig];
    }

    public function isImage($filePath){
            return @is_array(getimagesize($filePath)) ? true : false;
    }

    private function getTemplatePreview(Uploads $model){
            $filePath = PhotoLibrary::getUploadUrl().$model->ref.'/thumbnail/'.$model->real_filename;
            $isImage  = $this->isImage($filePath);
            if($isImage){
                $file = Html::img($filePath,['class'=>'file-preview-image', 'alt'=>$model->file_name, 'title'=>$model->file_name]);
            }else{
                $file =  "<div class='file-preview-other'> " .
                         "<h2><i class='glyphicon glyphicon-file'></i></h2>" .
                         "</div>";
            }
            return $file;
    }

    private function createThumbnail($folderName,$fileName,$width=250){
      $uploadPath   = PhotoLibrary::getUploadPath().'/'.$folderName.'/';
      $file         = $uploadPath.$fileName;
      $image        = Yii::$app->image->load($file);
      $image->resize($width);
      $image->save($uploadPath.'thumbnail/'.$fileName);
      return;
    }

    public function actionDeletefileAjax(){

        $model = Uploads::findOne(Yii::$app->request->post('key'));
        if($model!==NULL){
            $filename  = PhotoLibrary::getUploadPath().$model->ref.'/'.$model->real_filename;
            $thumbnail = PhotoLibrary::getUploadPath().$model->ref.'/thumbnail/'.$model->real_filename;
            if($model->delete()){
                @unlink($filename);
                @unlink($thumbnail);
                echo json_encode(['success'=>true]);
            }else{
                echo json_encode(['success'=>false]);
            }
        }else{
          echo json_encode(['success'=>false]);  
        }
    }
}

```

## เพิ่ม Widget Gallery
เพื่อให้สามารถดูรูปภาพที่เราได้อัพโหลดไปแล้ว เราจะทำการเรียกใช้งาน widget Gallery

โดยเพิ่มที่ไฟล์ `views\photo-library\view.php`เพิ่มการเรียกใช้งานเข้าไปด้านล่างสุด ซึ่งเราจะเรียกใช้งาน `getThumbnails()` ที่เราได้สร้างไว้แล้วตอนแก้ไข Model PhotoLibrary
```php
<div class="panel panel-default">
  <div class="panel-body">
     <?= dosamigos\gallery\Gallery::widget(['items' => $model->getThumbnails($model->ref,$model->event_name)]);?>
  </div>
</div>
```

## ลองใช้งาน

หน้าหลัก
![update](/images/upload-file/form-1.png)

การสร้างและแก้ไขข้อมูล
![update](/images/upload-file/form-2.png)

หน้า preview
![update](/images/upload-file/form-3.png)

เมื่อคลิกที่รูปภาพจะแสดงเป็น gallry
![update](/images/upload-file/form-4.png)
