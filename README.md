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

                //  把“ .”截取掉，
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

 












