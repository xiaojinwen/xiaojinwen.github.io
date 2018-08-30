---
title: 文件管理php
date: 2018-08-30 09:27:59
categories: ['后端'] 
tags: 后端
comments: true
---
### 有些代码不是自己写的 所以格式不太同
```
<?php
function fileAdir($dir) {
	if (!is_dir($dir)){
		mkdir($dir);
		chmod($dir,0777); 
		$data = scandir("$dir");    
		$allFile = array();        
		foreach ($data as $file) {
			if($file != '.' && $file != '..') {    //判断是否是文件夹内的文件夹
				$a = array(
					'file'=>$file,
					'isDir'=>is_dir($dir. '/' .$file),
					'name'=>$dir . '/' .$file
				);
				array_push($allFile, $a);
			}
		}
		return $allFile;
	} else{
		$data = scandir("$dir"); 
		$allFile = array();      
		foreach ($data as $file) {
			if($file != '.' && $file != '..') {    //判断是否是文件夹内的文件夹
				$a = array(
					'file'=>$file,
					'isDir'=>is_dir($dir. '/' .$file),
					'name'=>$dir . '/' .$file
				);
				array_push($allFile, $a);
			}
		}
		return $allFile;
	}
}
/**  读取文件类  **/
class file {
    public $file;
    public $filename;  //文件名字
    public $filetype;  //文件类型
    public $filesize;  //文件大小
    public $fileopen;  //打开文件
    public $fileread;  //读取文件

    //写入变量
    public $filepath;
    public $filecontent;   //保存文件用到的文件内容
    public $fileput;       //文件写入
	public function edits($files){
        $this->file = $files;
        if(!file_exists($this->file)){
            die("文件不存在！");
        }
        $this->fileopen = fopen($this->file, 'r');
        if(!$this->fileopen){
            die("文件读取失败");
        }
        //$this->fileread = htmlspecialchars($this->fileread);
        //开始获取文件的后缀名
        $filestr = strlen($this->file);
        $filepoint = strrpos($this->file, '.');
        $filesub = substr($this->file, $filepoint+1);
        $this->filetype = $filesub;
        $this->filename = basename($this->file);
		
		$this->filesize = filesize($this->file);
		if($this->filesize===0){
				return array(
				'filetype'=>$this->filetype,
				'filename'=>$this->filename,
				'filecontent'=>'',
				'filepath'=>$this->file
			);
			exit;
		}
        $this->fileread = fread($this->fileopen,$this->filesize);
		// 读取中文
		//$this->fileread =iconv('gb2312', 'utf-8', $this->fileread); 
        //以数组形式返回：文件类型 文件名称 文件内容 文件路径
        return array(
            'filetype'=>$this->filetype,
            'filename'=>$this->filename,
            'filecontent'=>$this->fileread,
            'filepath'=>$this->file
        );
        //关闭文件
        fclose($this->file);
    }
    /* 修改文件函数 */
    public function bc($filepath,$filecontent){
        $this->filepath = $filepath;
        //访问文件
        $this->fileopen = file_get_contents($this->filepath);
        if(!$this->fileopen){
            //die('文件打开失败');
        }
        //获取传进来的文件内容
        $this->filecontent = $filecontent;
        //反转义html
        //$this->filecontent = htmlspecialchars_decode($this->filecontent);
        //写入文件
        $this->fileput = file_put_contents($this->filepath, $this->filecontent);
        if($this->fileput){
            return '修改文件成功';
        }else {
            return '修改文件失败';
        }
    }
	// 创建文件夹
	public function create_folder($dir) {
        if (!file_exists($dir)){
            //mkdir($dir,0777,true);
            mkdir($dir);
			chmod($dir,0777); 
            return '创建文件夹成功';
        } else {
			return '创建文件夹已存在';
        }
    }
	// 创建文件
	public function create_file($dir) {
        if (!file_exists($dir)){
            $handle = fopen($dir, 'w');
			chmod($dir,0777); 
			if($handle){
				return '创建文件成功';
			} else{
				return '创建文件失败';
			}  
        } else {
            return '需创建的文件已经存在';
        }
    }
	// 重命名文件/文件夹
	public function fileRename($old_name,$new_name) {
        if(!file_exists($new_name)||file_exists($old_name)){
			$result = rename($old_name,$new_name);
			if($result){
				return	'重命名成功';
			} else{
				return	'重命名失败';
			}
        } else {
			return '目标文件已存在或原文件不存在';
        }
    }
	// 复制文件夹及文件夹下文件
	public function copyDirAndFile($src,$dst) {  // 原目录，复制到的目录
		if(file_exists($src)){
			$dir = opendir($src);
			@mkdir($dst);
			chmod($dst,0777); 
			while(false !== ($file = readdir($dir))) {
				if (( $file != '.' ) && ( $file != '..' )) {
					if (is_dir($src . '/' . $file)) {
						$this->copyDirAndFile($src . '/' . $file,$dst . '/' . $file);
						chmod($dst . '/' . $file,0777);
					} else {
						copy($src . '/' . $file,$dst . '/' . $file);
						chmod($dst . '/' . $file,0777); 
					}
				}
			}
			closedir($dir);
			return true;
		} else {
			return false;
		}
	}
	
	// 删除文件夹及文件夹下文件
	public function delDirAndFile($dirName){
		if(file_exists($dirName)){
			if ($handle = opendir("$dirName" )) {
			   while (false !== ($item=readdir($handle))) {
				   if ($item != "." && $item != "..") {
					   if (is_dir( "$dirName/$item")) {
							$this->delDirAndFile( "$dirName/$item" );
					   } else {
							if(unlink( "$dirName/$item" )){
								// echo "成功删除文件： $dirName/$item\n";
							}	
					   }
				   }
			   }
			   closedir( $handle );
			   if(rmdir( $dirName )) {
				   // echo "成功删除目录： $dirName\n";
			   }
			}
			return true;
		} else{
			return false;
		}
	}

	// 剪切文件夹及文件夹下文件
	public function cutDirAndFile($src,$dst) {  // 原目录，复制到的目录
		if(file_exists($src)){
			$dir = opendir($src);
			@mkdir($dst);
			chmod($dst,0777); 
			while(false !== ($file = readdir($dir))) {
				if (( $file != '.' ) && ( $file != '..' )) {
					if (is_dir($src . '/' . $file)) {
						$this->copyDirAndFile($src . '/' . $file,$dst . '/' . $file);
						chmod($dst . '/' . $file,0777);
					} else {
						copy($src . '/' . $file,$dst . '/' . $file);
						chmod($dst . '/' . $file,0777); 
						//unlink($src . '/' . $file);
					}
				}
			}
			$file = new file();
			$delete = $file->delDirAndFile($src);
			if(!$delete){
				return '文件夹删除失败';
			}
			$file = null;
			closedir($dir);
			return true;
		} else {
			return false;
		}
	}

	// 复制单个文件
	public function fileCopy($source_name,$target_name) {
        if(!file_exists($target_name)||file_exists($source_name)){
			//return copy($source_name,$target_name)?true:false;
			$result = copy($source_name,$target_name);
			chmod($target_name,0777); 
			return $result;
			if($result){
				return '复制文件成功';
			} else {
				return '复制文件失败';
			}
        } else {
            return '目标文件已经存在或者原始文件不存在';
        }
    }
	// 删除单个文件
	public function fileDelete($del_name) {
        if(file_exists($del_name)){
			$result = unlink($del_name);
			if($result){
				return '删除文件成功';
			} else {
				return '删除文件失败';
			}
        } else {
            return '要删除的文件不存在';
        }
    }
	// 剪切单个文件
	public function fileCut($source_name,$target_name) {
        if(!file_exists($target_name)||file_exists($source_name)){
			//return copy($source_name,$target_name)?true:false;
			$result = copy($source_name,$target_name);
			chmod($target_name,0777); 
			unlink($source_name);
			if($result){
				return '剪切文件成功';
			} else {
				return '剪切文件失败';
			}
        } else {
            return '目标文件已经存在或者原始文件不存在';
        }
    }
}
?>
```