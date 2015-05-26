# Relations & Virtual Attribute

ตัวอย่างง่ายๆ ในการสร้าง relation กับ model ตัวอย่างนี้ผมจำลองข้อมูลไว้ 2 ตารางคือ Patient & Hospital 
โดยที่ตาราง patient จะเกับรหัสของโรงพยาบาลไว้ คือ hospital_code

่จำลองง่ายๆ นะครับ

```
Patient
 - title
 - name
 - surname
 - hospital_code  --------+
                          |
Hospital                  |
 - code  -----------------+
 - name
 ```

เราจะทำการปรับปรุง `Model Patient` ให้สามารถเชื่อมกับ hospital เพื่อให้แสดงชื่อของโรงพยาบาลได้และสร้าง virtual attribute ชื่อ `hospitalName` จาก relation Hospital ด้วย เพื่อให้สามารถเรียกด้วยคำสั่งว่า `hospitalName` ได้ โดยไม่ต้องใช้ Hospital.name (!บางคนอาจสงสัยมันสั้นกว่าตรงใหน ฮา..)

อีกส่วนเราจะสร้าง virtual attribute ชื่อ  `$fullName` เพื่อแสดงข้อมูลชื่อ - นามสกุล โดยจะรวม โดยจะรวม name,surname ไว้เป็น field ใหม่เลยชื่อว่า fullName

> ผมใส่เครื่องหมาย `@` นำหน้าเพื่อให้มันไม่ error เวลาที่ไม่สามารถเชื่อมกับ `Hospital` เช่น  กรณีไม่ได้บันทึกระหัสโรงพยาบาลมา เวลาแสดงผลที่ `GridView` มันจะ error เพราะไม่สามารถเชื่อมได้ 

## เพิ่มใน Model Patient
```php
<?php
class Patient extends \yii\db\ActiveRecord{

	//.....

	// relation
	public  function getHospital(){
        return @$this->hasOne(Hospital::className(), ['code' => 'hospital_code']);
    }

    // virtual attribute hospitalName
    public function getHospitalName(){
        return @$this->hospital->name;
    }

    // virtual attribute fullName 
    public function getFullname(){

        return $this->title .$this->name. " " .$this->surname;
    }

    public function attributeLabels()
    {
        return [
        	//..............ฟิวด์อื่นๆ

           'fullName' => Yii::t('app', 'ชื่อ-นามสกุล'),
	       'hospitalName' => Yii::t('app', 'โรงพยาบาล'),

            //..............
        ];
    }

```
## การเรียกใช้ใน Gridview 

```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    'columns' => [
        ['class' => 'yii\grid\SerialColumn'],
        'title',
        'name',
        'surname',
        //เรียกผ่าน relation โดยตรง
        'hospital.name',
        // เรียกผ่าน virtual attribute hospitalName
        'hospitalName',
        // เรียกผ่าน virtual attribute fullName
        'fullName'
    ],
]); ?>
```

# การ Sort 

ปกติเราสามารถคลิก sort `asc,desc` ได้จากหัว column ใน `GridView ` ได้ แต่ถ่าหากเป็นฟิวด์ที่เราเพิ่มขึันมาเช่น hospitalName, fullName นั้นจะไม่สามารถใช้งานได้เลย เราจะต้องปรับแต่งเพิ่มเิตมนิดหน่อย

โดยการสั่งให้มันทำการ sort ตาม column ที่เรากำหนดได้ เราจะทำการเพิ่มในไฟล์ `PatientSearch.php` ปกติมันจะมีมาพร้อมกับตอน crud อยู่แล้ว ทำการแก้ไขที่` function Search()`

```php
<?php
class PatientSearch extends Patient {

	///.................
    public function search($params)
    {
        $query = Patient::find();

        $query->joinWith(['hospital']);

        $dataProvider = new ActiveDataProvider([
            'query' => $query,
        ]);

        // สำหรับ coluumn hospitalName
        $dataProvider->sort->attributes['hospitalName'] = [
            'asc' => ['lib_hospital.name' => SORT_ASC],
            'desc' => ['lib_hospital.name' => SORT_DESC],
        ];

		// สำหรับ coluumn fullName
         $dataProvider->sort->attributes['fullName'] = [
            'asc' => ['name' => SORT_ASC, 'surname' => SORT_ASC],
            'desc' => ['name' => SORT_DESC, 'surname' => SORT_DESC],
            'label' => 'ชื่อ - นามสกุล',
            'default' => SORT_ASC
        ];

        //................
     }

```
หลังจากนั้นเราจะทำการคลิก sort ที่หัวคอลัมน์ได้เลย สะดวกมากๆ ^^


 # การค้นหา

 เราสามารถปรับแต่งให้คอลัมน์ hospitalName,fullName ที่เราเพิ่มขึน้มาใหม่นันสามารถค้นหาได้ โดยเพิ่มที่ PatientSearch.php ที่ function Search()

 เราจะสามารถใช้ฟิวด์ที่เรากำหนดเองได้

```php
<?php
		class PatientSearch extends Patient
		{

		    public $hospitalName;
		    public $fullName;	

			public function rules()
		    {
		        return [
		            // ..........
		            [hospitalName','fullName'], 'safe'],
		        ];
		    }

		    public function search($params)
		    {



		       //..........

	            $query->orFilterWhere(['like', 'patient.name', $this->fullName])
	                  ->orFilterWhere(['like', 'patient.surname', $this->fullName])
	                  ->orFilterWhere(['like', 'lib_hospital.name', $this->hospitalName]);
		    }

		    //.........
    }
```

ขอให้สนุกกับการเรียนรู้ครับ ....
อ่านต่อเพิ่มเติมได้ที่นี่ครับ
- [Yii 2.0: Filter & Sort by calculated/related fields in GridView Yii 2.0 ](http://www.yiiframework.com/wiki/621/filter-sort-by-calculated-related-fields-in-gridview-yii-2-0/)
- [Yii 2.0: Displaying, Sorting and Filtering Model Relations on a GridView](http://www.yiiframework.com/wiki/653/displaying-sorting-and-filtering-model-relations-on-a-gridview/)











