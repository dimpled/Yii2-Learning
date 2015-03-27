## อัพโหลดไฟล์แบบ Ajax

เราจะเพิ่มการอัพโหลดแบบ ajax คุณสมบัติหลักคือ
- สามารถอัพโหลดได้ทันทีผ่าน  ajax  โดยไม่ต้องรอ create record
- สามารถกดปุ่ม upload ajax ได้ หรือ กด submit ผ่านฟอร์มได้เช่นกัน
- resize image ถ้าเป็นรูปภาพและสร้าง thumbnail
- สามารถกดปุ่มลบรายการที่อัพโหลดได้


เราจะทำการสร้าง Uploads() ฟังก์ชันนี้มีหน้าทีรับไฟล์จาก ajax from แล้วส่งข้อมูลมาทำการอัพโหลดไฟล์ สามารถอัพได้ทั้งภาพและไฟล์ ถ้าเป็นภาพจะทำการ resize และสร้าง thumbnail ให้

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

### สร้าง actionUploadAjax
เป็น function ที่เอาไว้รองรับการอัพโหลดไฟล์ผ่าน ajax ซึ่งเราจะใช้ Upload() ที่เราสร้างไว้ โดยรับค่า parameter 1 ตัวคือ $isAjax เพื่อไปเช็คว่าเป็นการอัพโหลดผ่าน ajax หรือผ่านการ submission form จากนั้นให้ทดลองทำการอัพโหลดไฟล์ลองดู
```php
public function actionUploadAjax(){
       $this->Uploads(true);
}
```

### เรียกใช้งานที่ actionCreate และ actionUpdate
ปรับปรุงให้ action เดิมๆ สามารถรองรับการอัพโหลดจากฟอร์ม ajax ได้ เพราะหากเค้าไม่คลิกปุ่มอัพโหลด ajax แล้วข้อมูลจะยังไม่ถูกอัพโหลด เราจะแก้ไขโดยการเรียกใช้งาน function Upload() ที่เราได้สร้างไว้และใช้กับ actionUploadAjax() ซึ่งสามารถนำมาใช้กับ acionCreate และ  actionUpdate ได้

โดยเพิ่ม `this->Uploads(false);`เข้าไป
```php
public function actionCreate()
{
    $model = new Freelance();

    if ($model->load(Yii::$app->request->post()) ) {

        $this->CreateDir($model->ref);

        $this->Uploads(false); //<---

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

ส่วน actionUpdate() เพิ่ม getInitialPreview() เข้ามาด้วยเพื่อใช้แสดง thumbnail ในตัว ajax upload
```php
public function actionUpdate($id)
 {
     $model = $this->findModel($id);

     list($initialPreview,$initialPreviewConfig) = $this->getInitialPreview($model->ref);  //<----

     $tempCovenant = $model->covenant;
     $tempDocs     = $model->docs;
     if ($model->load(Yii::$app->request->post())) {

         $this->Uploads(false); //<----

         $model->covenant = $this->uploadSingleFile($model,$tempCovenant);
         $model->docs = $this->uploadMultipleFile($model,$tempDocs);
         if($model->save()){
             return $this->redirect(['view', 'id' => $model->id]);
         }
     }

     return $this->render('update', [
         'model' => $model,
         'initialPreview'=>$initialPreview, //<----
         'initialPreviewConfig'=>$initialPreviewConfig ///<----
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
