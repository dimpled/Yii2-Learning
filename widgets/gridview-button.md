# เพิ่ม Botton Group ใน GridView

![](/images/grid-button.png)

```php
<?= GridView::widget([
     'dataProvider' => $dataProvider,
     'filterModel' => $searchModel,
     'columns' => [
         ['class' => 'yii\grid\SerialColumn'],
         'id',
         'country_code',
         'country_name',
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

rr
