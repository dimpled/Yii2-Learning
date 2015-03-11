# เพิ่ม Botton Group ใน GridView

เห็น button group ใน bootstrap สวนดี ก็เลยจับมาใส่ใน actionColumn  ซักหน่อย ของเดิมดูยาก บางทีมองแทบไม่ออกว่ามันคือปุ่ม
![](/images/grid-button.png)

ง่ายมากๆ เพิ่ม div ใน property `template` และใส่ class `btn-group btn-group-sm` ครอบปุ่มต่างๆไว้ และเพิ่ม class `btn btn-default` ที่ link
> จริงๆ ใน version 2.4 จะมี buttonOptions ให้สามารถคอนฟิกได้เลยสั้นๆ ตอนนี้รอมันออก 2.4 ค่อยอัพเดทไปใช้ ตอนนี้ใช้แบบนี้ไปก่อน
```php
<?= GridView::widget([
     'dataProvider' => $dataProvider,
     'filterModel' => $searchModel,
     'columns' => [
         ['class' => 'yii\grid\SerialColumn'],
         'id',
         'country_code',
         'country_name',
         //......
         [
             'class' => 'yii\grid\ActionColumn',
             'options'=>['style'=>'width:120px;'],
             'template'=>'<div class="btn-group btn-group-sm" role="group" aria-label="...">{view}{update}{delete}</div>',
             'buttons'=>[
                 'view'=>function($url,$model,$key){
                     return Html::a('<i class="glyphicon glyphicon-eye-open"></i>',$url,['class'=>'btn btn-default']);
                 },
                 'update'=>function($url,$model,$key){
                     return Html::a('<i class="glyphicon glyphicon-pencil"></i>',$url,['class'=>'btn btn-default']);
                 },
                 'delete'=>function($url,$model,$key){
                      return Html::a('<i class="glyphicon glyphicon-trash"></i>', $url,[
                             'title' => Yii::t('yii', 'Delete'),
                             'data-confirm' => Yii::t('yii', 'Are you sure you want to delete this item?'),
                             'data-method' => 'post',
                             'data-pjax' => '0',
                             'class'=>'btn btn-default'
                             ]);
                 }
             ]
         ],
     ],
 ]); ?>
```
