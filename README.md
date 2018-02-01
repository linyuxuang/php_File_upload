# php_File_upload
php文件上传
 
 
     文件上传
 
  	1. 单个文件上传
  	2. 多个文件上传
 
 	一、PHP配置文件中和上传文件有关的选项
   
            file_uploads = on 
            upload_max_filesize= 200M  最大不要超过服务器的内存
            upload_tmp_dir = c:/uploads/
            post_max_size = 250M
 
      二、上传表单需要的注意事项
 
             1. 如果有文件上传操作表单的提交方法必须 HTTP post 
             2. 表单上传需要使用type为file的表
             3. enctype="multipart/form-data" 只有文件上传时才使用这个值 ，
             用来指定表单编码的数据方式， 让服务器知道，我们要传递一个文件并带有常规的表单信息。
            4. 建议添加一个 MAX_FILE_SIZE 隐藏表单， 值的单位也是字节
 
 
    三、PHP处理上传的数据
    
          $_POST 接收非上传的数据
          如果是文件上传的数据则使用 $_FILES处理上传的文件

 
 

   index.html  文件
 
 
       	<form action="file.php" method="post" enctype="multipart/form-data">
			
            商品名字：<input type="text" name="spName"><br/>

            商品价格：<input type="text" name="jiage"><br /> 

            商品数量：<input type="text" name="shumu"><br />
                  <input type="hidden" name="MAX_FILE_SIZE" value="1000000">

            上传商品文件: <input type="file" name="pic"><br />		


            <input type="submit" name="sub"  value="添加商品">		

            </form>



   file.php  文件
       
       
       
                 //	$_FILES 是接受文件上传的
                // $_POST  是接受非文件上传数据(如，用户名，密码，等)


            1> 使用$_FILES['imgs']["error"] 检查错误

               if($_FILES["pic"]["error"]>0){

               switch($_FILES["pic"]["error"]){
                  case 1:
                          echo "上传的文件超过了 php.ini 中 upload_max_filesize 选项限制的值<br>";
                              break;
                  case 2:
                             echo "上传文件的大小超过了 HTML 表单中 MAX_FILE_SIZE 选项指定的值";
                             break;
                  case 3:
                            echo "文件只有部分被上传";
                           break;
                  case 4:
                            echo "没有文件被上传";
                            break;
                  default:
                       echo "末知错误"; 
                  }
                        exit;

             }

             2>  使用$_FILES["pic"]["size"] 限制大小 单位字节 1M=1000000字节

              $maxsize=50000;    //50k
             if($_FILES["pic"]["size"]>$maxsize){
               echo "上传的文件不能超过".$maxsize."字节";
               exit;
             }

             //  这样可以获取到文件名
             //   echo $_FILES["pic"]["name"];

                //  以“ . ”分割成数组，
                 //explode — 使用一个字符串分割另一个字符串
                 $arr= explode(".",$_FILES["pic"]["name"]) ;

                $allowtype=array("png", "gif", "jpg", "jpeg"); 

                   //这样拿到文件最后的后缀名  
                 $hz=$arr[count($arr)-1];

                 //in_array — 检查数组中是否存在某个值
                   if(!in_array($hz,$allowtype)){
                        echo "这是不允许的后缀名（类型）";
                        exit; 
                   }

           4 >  将让传后的文件名改名

          $filepath="./uploads/";  // 存问文件目录名
           // 把上传的文件名做成  随机的文件名
             //rand — 产生一个随机整数
            $randname=date("Y").date("m").date("d").date("H").date("i").date("s").".".$hz;



                //5  >  将临时位置的文件移动到指定的目录上即可

               //  is_uploaded_file — 判断文件是否是通过 HTTP POST 上传的
               //  tmp_name  这是点击提交后  存放的临时文件，
               //  move_uploaded_file — 将上传的文件移动到新位置


        //  这个是把上传的文件名没有变，存放到指定的目录下
          // if(is_uploaded_file($_FILES["pic"]["tmp_name"])){  
          //  if(move_uploaded_file($_FILES["pic"]["tmp_name"],"./uploads/".$_FILES["pic"]["name"])){
          //     	   	     echo "上传成功";
          //     	      }else{
          //     	      	echo "上传失败";
          //     	   } 
          //     }


            //把上传的文件名改成随机文件名  存放在指定的目录
              if(is_uploaded_file($_FILES["pic"]["tmp_name"])){  
                      if(move_uploaded_file($_FILES["pic"]["tmp_name"], $filepath.$randname)){
                            echo "上传成功";
                         }else{
                          echo "上传失败";
                      } 
               }else{
                echo "这不是上传文件";
               }

 

封装成类的文件上传

        
	 单个文件上传
	    
	 form.html文件
	     
	     <form action="upload.php" method="post" enctype="multipart/form-data">
		<input type="hidden" name="MAX_FILE_SIZE" value="100000000">
		<input type="file" name="spic[]"> <br>
		<input type="file" name="spic[]"> <br>
		<input type="file" name="spic[]"> <br>
		<input type="file" name="spic[]"> <br>

		<input type="submit" name="sub" value="upload file"><br>
	     </form>	

	 
	 FileUpload_1.class文件
	  
	 <?php
	  class FileUpload {
		private $filepath;     //指定上传文件保存的路径
		private $allowtype=array('gif', 'jpg', 'png', 'jpeg');  //充许上传文件的类型
		private $maxsize=1000000;  //允上传文件的最大长度 1M
		private $israndname=true;  //是否随机重命名， true false不随机，使用原文件名
		private $originName;   //源文件名称
		private $tmpFileName;   //临时文件名
		private $fileType;  //文件类型
		private $fileSize;  //文件大小
		private $newFileName; //新文件名
		private $errorNum=0;  //错误号
		private $errorMess=""; //用来提供错误报告



		//用于对上传文件初使化
		//1. 指定上传路径， 2，充许的类型， 3，限制大小， 4，是否使用随机文件名称
		//让用户可以不用按位置传参数，后面参数给值不用将前几个参数也提供值
		function __construct($options=array()){
			foreach($options as $key=>$val){
				$key=strtolower($key);
				//查看用户参数中数组的下标是否和成员属性名相同
				if(!in_array($key,get_class_vars(get_class($this)))){
					continue;
				}

				$this->setOption($key, $val);
			}
		 
		
		}
	


	 private function getError(){
		$str="上传文件<font color='red'>{$this->originName}</font>时出错：";

		switch($this->errorNum){
			case 4: $str .= "没有文件被上传"; break;
			case 3: $str .= "文件只被部分上传"; break;
			case 2: $str .= "上传文件超过了HTML表单中MAX_FILE_SIZE选项指定的值"; break;
			case 1: $str .= "上传文件超过了php.ini 中upload_max_filesize选项的值"; break;
			case -1: $str .= "末充许的类型"; break;
			case -2: $str .= "文件过大，上传文件不能超过{$this->maxSize}个字节"; break;
			case -3: $str .= "上传失败"; break;
			case -4: $str .= "建立存放上传文件目录失败，请重新指定上传目录"; break;
			case -5: $str .= "必须指定上传文件的路径"; break;

			default: $str .=  "末知错误";
		}

			return $str.'<br>';
		}
	
		//用来检查文件上传路径
		private function checkFilePath(){
			if(empty($this->filepath)) {
				$this->setOption('errorNum', -5);
				return false;
			}

			if(!file_exists($this->filepath) || !is_writable($this->filepath)){
				if(!@mkdir($this->filepath, 0755)){
					$this->setOption('errorNum', -4);
					return false;
				}
			}
			return true;
		}
		//用来检查文件上传的大小
		private function checkFileSize() {
			if($this->fileSize > $this->maxsize){
				$this->setOPtion('errorNum', '-2');
				return false;
			}else{
				return true;
			}
		}

		//用于检查文件上传类型
		private function checkFileType() {
			if(in_array(strtolower($this->fileType), $this->allowtype)) {
				return true;
			}else{
				$this->setOption('errorNum', -1);
				return false;
			}
		}
		//设置上传后的文件名称
		private function setNewFileName(){
			if($this->israndname){
				$this->setOption('newFileName', $this->proRandName());
			} else {
				$this->setOption('newFileName', $this->originName);
			}
		}



		//设置随机文件名称
		private function proRandName(){
			$fileName=date("YmdHis").rand(100,999);
			return $fileName.'.'.$this->fileType;
		}
	
		private function setOption($key, $val){
			$this->$key=$val;
		}
		//用来上传一个文件
		function uploadFile($fileField){
			$return=true;
			//检查文件上传路径
			if(!$this->checkFilePath()){
				$this->errorMess=$this->getError();
				return false;
			}

			
			$name=$_FILES[$fileField]['name'];
			$tmp_name=$_FILES[$fileField]['tmp_name'];
			$size=$_FILES[$fileField]['size'];
			$error=$_FILES[$fileField]['error'];

				
				if($this->setFiles($name, $tmp_name, $size, $error)){
					if($this->checkFileSize() && $this->checkFileType()){
							$this->setNewFileName();

							if($this->copyFile()){
								return true;
							}else{
								$return=false;
							}
								
						}else{
							$return=false;
						}	
					}else{
						$return=false;
					}
					
					

					if(!$return)
						$this->errorMess=$this->getError();


					return $return;
			
		}

		private function copyFile(){
			if(!$this->errorNum){
				$filepath=rtrim($this->filepath, '/').'/';
				$filepath.=$this->newFileName;

				if(@move_uploaded_file($this->tmpFileName, $filepath))	{
					return true;
				}else{
					$this->setOption('errorNum', -3);
					return false;
				}
					
			}else{
				return false;
			}
		}

		//设置和$_FILES有关的内容
		private function setFiles($name="", $tmp_name='', $size=0, $error=0){
		
			$this->setOption('errorNum', $error);
				
			if($error){
				return false;
			}

			$this->setOption('originName', $name);
			$this->setOption('tmpFileName', $tmp_name);
			$arrStr=explode('.', $name); 
			$this->setOption('fileType', strtolower($arrStr[count($arrStr)-1]));
			$this->setOption('fileSize', $size);	

			return true;
		}	

		//用于获取上传后文件的文件名
		function getNewFileName(){
			return $this->newFileName;
		}
		//上传如果失败，则调用这个方法，就可以查看错误报告
		function getErrorMsg() {
			return $this->errorMess;
		}
	}

	 
	
	
    upload.php文件	
	
	
		  <?php
		require "FileUpload1.class.php";

		$up=new FileUpload(array('isRandName'=>true,'allowType'=>array('txt', 'doc', 'php', 'gif'),'FilePath'=>'./uploads/', 'MAXSIZE'=>200000));

		echo '<pre>';

		if($up->uploadFile('spic')){
			print_r($up->getNewFileName());
		}else{
			print_r($up->getErrorMsg());	
		}

		echo '</pre>';




单个文件上传又可以多个文件上传


          
            <?php
	class FileUpload {
		private $filepath;     //指定上传文件保存的路径
		private $allowtype=array('gif', 'jpg', 'png', 'jpeg');  //充许上传文件的类型
		private $maxsize=1000000;  //允上传文件的最大长度 1M
		private $israndname=true;  //是否随机重命名， true false不随机，使用原文件名
		private $originName;   //源文件名称
		private $tmpFileName;   //临时文件名
		private $fileType;  //文件类型
		private $fileSize;  //文件大小
		private $newFileName; //新文件名
		private $errorNum=0;  //错误号
		private $errorMess=""; //用来提供错误报告



		//用于对上传文件初使化
		//1. 指定上传路径， 2，充许的类型， 3，限制大小， 4，是否使用随机文件名称
		//让用户可以不用按位置传参数，后面参数给值不用将前几个参数也提供值
		function __construct($options=array()){
			foreach($options as $key=>$val){
				$key=strtolower($key);
				//查看用户参数中数组的下标是否和成员属性名相同
				if(!in_array($key,get_class_vars(get_class($this)))){
					continue;
				}

				$this->setOption($key, $val);
			}
		 
		
		}
	

		private function getError(){
		  $str="上传文件<font color='red'>{$this->originName}</font>时出错：";

		switch($this->errorNum){
			case 4: $str .= "没有文件被上传"; break;
			case 3: $str .= "文件只被部分上传"; break;
			case 2: $str .= "上传文件超过了HTML表单中MAX_FILE_SIZE选项指定的值"; break;
			case 1: $str .= "上传文件超过了php.ini 中upload_max_filesize选项的值"; break;
			case -1: $str .= "末充许的类型"; break;
			case -2: $str .= "文件过大，上传文件不能超过{$this->maxSize}个字节"; break;
			case -3: $str .= "上传失败"; break;
			case -4: $str .= "建立存放上传文件目录失败，请重新指定上传目录"; break;
			case -5: $str .= "必须指定上传文件的路径"; break;

			default: $str .=  "末知错误";
		}

		return $str.'<br>';
		}
	
		//用来检查文件上传路径
		private function checkFilePath(){
			if(empty($this->filepath)) {
				$this->setOption('errorNum', -5);
				return false;
			}

			if(!file_exists($this->filepath) || !is_writable($this->filepath)){
				if(!@mkdir($this->filepath, 0755)){
					$this->setOption('errorNum', -4);
					return false;
				}
			}
			return true;
		}
		//用来检查文件上传的大小
		private function checkFileSize() {
			if($this->fileSize > $this->maxsize){
				$this->setOPtion('errorNum', '-2');
				return false;
			}else{
				return true;
			}
		}

		//用于检查文件上传类型
		private function checkFileType() {
			if(in_array(strtolower($this->fileType), $this->allowtype)) {
				return true;
			}else{
				$this->setOption('errorNum', -1);
				return false;
			}
		}
		//设置上传后的文件名称
		private function setNewFileName(){
			if($this->israndname){
				$this->setOption('newFileName', $this->proRandName());
			} else {
				$this->setOption('newFileName', $this->originName);
			}
		}



		//设置随机文件名称
		private function proRandName(){
			$fileName=date("YmdHis").rand(100,999);
			return $fileName.'.'.$this->fileType;
		}
	
		private function setOption($key, $val){
			$this->$key=$val;
		}
		//用来上传一个文件
		function uploadFile($fileField){
			$return=true;
			//检查文件上传路径
			if(!$this->checkFilePath()){
				$this->errorMess=$this->getError();
				return false;
			}

			
			$name=$_FILES[$fileField]['name'];
			$tmp_name=$_FILES[$fileField]['tmp_name'];
			$size=$_FILES[$fileField]['size'];
			$error=$_FILES[$fileField]['error'];

			if(is_Array($name)){
			   $errors=array();

			  for($i=0; $i<count($name); $i++){
			   if($this->setFiles($name[$i], $tmp_name[$i], $size[$i], $error[$i])){
					if(!$this->checkFileSize() || !$this->checkFileType()){
						$errors[]=$this->getError();
						$return=false;
					}
				}else{
					$error[]=$this->getError();
					$return=false;
				}

				if(!$return)
					$this->setFiles();
			}

		    if($return){
		      $fileNames=array();

		     for($i=0; $i<count($name); $i++){
			if($this->setFiles($name[$i], $tmp_name[$i], $size[$i], $error[$i])){
				$this->setNewFileName();
				if(!$this->copyFile()){
					$errors=$this->getError();
					$return=false;
				}else{
					$fileNames[]=$this->newFileName;
				}
			}
		}

					$this->newFileName=$fileNames;
				}

				$this->errorMess=$errors;
				return $return;
			} else {
				
					if($this->setFiles($name, $tmp_name, $size, $error)){
						if($this->checkFileSize() && $this->checkFileType()){
							$this->setNewFileName();

							if($this->copyFile()){
								return true;
							}else{
								$return=false;
							}
								
						}else{
							$return=false;
						}	
					}else{
						$return=false;
					}
					
					

					if(!$return)
						$this->errorMess=$this->getError();


					return $return;
			}			
		}

		private function copyFile(){
			if(!$this->errorNum){
				$filepath=rtrim($this->filepath, '/').'/';
				$filepath.=$this->newFileName;

				if(@move_uploaded_file($this->tmpFileName, $filepath))	{
					return true;
				}else{
					$this->setOption('errorNum', -3);
					return false;
				}
					
			}else{
				return false;
			}
		}

		//设置和$_FILES有关的内容
		private function setFiles($name="", $tmp_name='', $size=0, $error=0){
		
			$this->setOption('errorNum', $error);
				
			if($error){
				return false;
			}

			$this->setOption('originName', $name);
			$this->setOption('tmpFileName', $tmp_name);
			$arrStr=explode('.', $name); 
			$this->setOption('fileType', strtolower($arrStr[count($arrStr)-1]));
			$this->setOption('fileSize', $size);	

			return true;
		}	

		//用于获取上传后文件的文件名
		function getNewFileName(){
			return $this->newFileName;
		}
		//上传如果失败，则调用这个方法，就可以查看错误报告
		function getErrorMsg() {
			return $this->errorMess;
		}
	}







