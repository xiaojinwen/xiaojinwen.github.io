---
title: 七牛云上传文件demo
date: 2018-09-20 16:46:46
categories: ['前端'] 
tags: 前端
comments: true
---
## 七牛云上传文件demo

这是我前几天写的 因为之前的上传会出现文件的MimeType不对 还有一些错误 所以现在写的有些复杂  (删除七牛云上与本地相同的文件-这一步其实可以没有,但我担心有重复文件导致不上传,所以还是删除吧)
> 读取本地保存的上一次上传记录
> 删除七牛云上一次上传记录的文件
> 删除七牛云上与本地相同的文件
> 上传文件并指定文件的MimeType
> 记录上传的文件


```bash
const fs = require('fs')
const qiniu = require('qiniu')

// 授权秘钥
const accessKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
const secretKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'

// 存储空间名称
const bucket = 'xiaojw'

// 要上传的资源目录
const staticPath = '../xxx_trunk/xiaojw'


// 上传后的文件前缀
const prefix = 'xiaojw'


const cdnPrefix = 'https://cdn.xxxxxxxx.com/'

// 创建鉴权对象
const mac = new qiniu.auth.digest.Mac(accessKey, secretKey)

// 创建并修改配置对象(Zone_z0=华东 Zone_z1=华北 Zone_z2=华南 Zone_na0=北美)
const config = new qiniu.conf.Config()
config.zone = qiniu.zone.Zone_z2

// 创建额外内容对象
const putExtra = new qiniu.form_up.PutExtra()

// 创建表单上传对象
const formUploader = new qiniu.form_up.FormUploader(config)

// 文件批量操作对象
const bucketManager = new qiniu.rs.BucketManager(mac, config);

const Mime = ['application/javascript', 'text/css', 'image/jpeg', 'image/png', 'text/html']


// 文件上传方法
function uploadFile(localFile) {
  // 配置上传到七牛云的完整路径
  const key = localFile.replace(staticPath, prefix)
  // console.log('准备上传文件: ' + key)

  const options = {
    scope: bucket + ":" + key
  }
  const putPolicy = new qiniu.rs.PutPolicy(options)
  // 生成上传凭证
  const uploadToken = putPolicy.uploadToken(mac)

  // 上传文件
  let index = key.lastIndexOf('.')
  if (index) {
    let suffix = key.slice(index + 1)
    switch (suffix) {
      case 'js':
        putExtra.mimeType = Mime[0]
        break
      case 'css':
        putExtra.mimeType = Mime[1]
        break
      case 'jpeg':
      case 'jpg':
        putExtra.mimeType = Mime[2]
        break
      case 'png':
        putExtra.mimeType = Mime[3]
        break
      case 'html':
        putExtra.mimeType = Mime[4]
        break
      default:
        console.log('未设置的上传类型')
    }
  }
  // console.log(key)
  if (!key) {
    console.log('路径出问题')
    return
  }
  formUploader.putFile(uploadToken, key, localFile, putExtra, function (respErr, respBody, respInfo) {
    if (respErr) {
      console.log('上传出错:\t' + cdnPrefix + key)
      console.log(respErr)
      // throw respErr
      return
    }
    // console.log('已上传: ', respBody.key)
    appendFile(`${respBody.key}\n`)
    // appendFile(`${key}\n`)
  })
}

// 目录上传方法
async function uploadDirectory(dirPath) {
  fs.readdir(dirPath, function (err, files) {
    if (err) throw err
    // 遍历目录下的内容
    files.forEach(item => {
      let path = `${dirPath}/${item}`
      fs.stat(path, function (err, stats) {
        if (err) throw err
        // 是目录就接着遍历 否则上传
        if (stats.isDirectory()) uploadDirectory(path)
        else {
          const key = path.replace(staticPath, prefix)
          // 删除七牛云上与本地相同的文件
          // console.log(key)
          // console.log(path)
          deleteSigleF(key).then(()=>{
            uploadFile(path)
          }).catch(()=>{
            uploadFile(path)
          })
        }
      })
    })
  })
}

// 批量删除文件方法
function deleteFile(deleteOperations) {
  return new Promise((resolve, reject) => {
    //每个operations的数量不可以超过1000个，如果总数量超过1000，需要分批发送
    if (!deleteOperations) {
      console.log('删除的文件数组不存在')
      return false
    }
    bucketManager.batch(deleteOperations, function (err, respBody, respInfo) {
      if (err) {
        console.log("文件批量删除出错");
        console.log(err);
        // throw err;
      } else {
        // 200 is success, 298 is part success
        if (parseInt(respInfo.statusCode / 100) == 2) {
          respBody.forEach(function (item) {
            if (item.code == 200) {
              // console.log(item.code + "\tsuccess")
              console.log("删除成功")
              // console.log(respBody)
              resolve("删除成功")
            } else {
              // console.log(item.code + "\t" + item.data.error)
              console.log("删除失败" + item.data.error)
              reject("删除失败" + item.data.error)
            }
          });
        } else {
          console.log(respInfo.deleteusCode);
          console.log(respBody);
          console.log('错误101--------------------');
          reject(respBody)
        }
      }
    })
  })
}

// 删除单个文件
function deleteSigleF(path) {
  return new Promise((resolve, reject) => {
    bucketManager.delete(bucket, path, function (err, respBody, respInfo) {
      if (err) {
        // console.log("删除失败" + err + "\t 路径:" + path);
        console.log("删除失败" + err);
        reject()
        //throw err;
      } else {
        // console.log(respInfo.statusCode, respBody);
        console.log(respBody, path)
        resolve()
      }
    })
  })
}

async function readFile(returnType = false) {
  return new Promise((resolve, reject) => {
    fs.readFile('./file.txt', {flag: 'r+', encoding: 'utf-8'}, function (err, data) {
      if (err) {
        reject(err)
      } else {
        let dataArr = data.split('\n')
        if (returnType) {
          resolve(dataArr)
          return
        }
        console.log('\n上一次上传文件--------------------\n')
        console.log(dataArr)
        let deleteOperations = []
        dataArr.forEach((item) => {
          if (item) {
            deleteOperations.push(qiniu.rs.deleteOp(bucket, item))
          }
        })
        resolve(deleteOperations)
      }
    })
  })
}

function writeFile(content) {
  fs.writeFile('./file.txt', content, {flag: 'w', encoding: 'utf-8', mode: '0666'}, function (err) {
    if (err) {
      console.log("文件写入失败")
    } else {
      console.log("删除服务器上相同文件,记录文件" + (content ? '变为' + content + '\n' : '清空,---------------\n'));
    }
  })
}

function appendFile(content) {
  fs.writeFile('./file.txt', content, {flag: 'a', encoding: 'utf-8', mode: '0666'}, function (err) {
    if (err) {
      console.log("已上传但未记录:\t" + cdnPrefix + content.replace('\n', ''))
    } else {
      console.log("已上传并记录:\t" + cdnPrefix + content.replace('\n', ''));
    }
  })
}

function uploadHome() {
  // 清空文件内容
  writeFile('')
  fs.exists(staticPath, function (exists) {
    if (!exists) {
      console.log('目录不存在！')
    } else {
      console.log('\n开始上传...')
      uploadDirectory(staticPath)
    }
  })
}

// 读取本地保存的上一次上传记录
// 删除七牛云上一次上传记录的文件
// 删除七牛云上与本地相同的文件
// 上传文件并指定文件的MimeType
// 记录上传的文件

async function exec() {
  // 读取
  let deleteOperations = await readFile()
  // console.log(deleteOperations)
  // 删除七牛云上一次上传记录的文件
  console.log('\n删除上一次上传文件---------------\n')
  if (deleteOperations.length) {
    // let deleteOP = await deleteFile(deleteOperations)
    // console.log(deleteOP)
    deleteFile(deleteOperations).then((res) => {
      // console.log(res)
      uploadHome()
    }).catch((err) => {
      // console.log(err)
      uploadHome()
    })
  } else {
    uploadHome()
  }
}

exec()

```
运行 
`node qiniuUpload.js`

## 修改
```bash
const fs = require('fs');
const qiniu = require('qiniu');
// 授权秘钥
const accessKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';
const secretKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';
// 存储空间名称
const bucket = "xiaojw";

const ENV = process.argv[2] === 'prod' ? 'prod' : (process.argv[2] === 'test' ? 'test' : 'dev');
const date = new Date();
const appendPath = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;

// 要上传的资源目录
const staticPath = `./dist`;

// 上传后的文件前缀
const prefix = `admin/${ENV}/${appendPath}`;
// cdn域名
const cdnPrefix = 'https://cdn.xxxxx.com/';

// 创建鉴权对象
const mac = new qiniu.auth.digest.Mac(accessKey, secretKey);

// 创建并修改配置对象(Zone_z0=华东 Zone_z1=华北 Zone_z2=华南 Zone_na0=北美)
const config = new qiniu.conf.Config();
config.zone = qiniu.zone.Zone_z2;

// 创建额外内容对象
const putExtra = new qiniu.form_up.PutExtra();

// 创建表单上传对象
const formUploader = new qiniu.form_up.FormUploader(config);

// 文件批量操作对象
const bucketManager = new qiniu.rs.BucketManager(mac, config);

const Mime = {
	js: 'application/javascript',
	css: 'text/css',
	jpeg: 'image/jpeg',
	jpg: 'image/jpeg',
	png: 'image/png',
	html: 'text/html',
	gz: 'application/x-gzip',
	gif: 'image/gif'
};
const MineKeyArr = Object.keys(Mime);

// 文件上传方法
function uploadFile(localFile) {
	// 配置上传到七牛云的完整路径
	const key = localFile.replace(staticPath, prefix);
	const options = {
		scope: bucket + ":" + key
	};
	const putPolicy = new qiniu.rs.PutPolicy(options);
	// 生成上传凭证
	const uploadToken = putPolicy.uploadToken(mac);

	// 上传文件
	let index = key.lastIndexOf('.');
	if (index) {
		let suffix = key.slice(index + 1);
		if (MineKeyArr.includes(suffix)) {
			putExtra.mimeType = Mime[suffix];
		} else {
			console.log('未设置的上传类型');
		}
	}
	if (!key) {
		console.log('路径出问题');
		return;
	}
	formUploader.putFile(uploadToken, key, localFile, putExtra, function (respErr, respBody, respInfo) {
		if (respErr) {
			console.log('上传出错:\t' + cdnPrefix + key);
			console.log(respErr);
			return;
		}
		baseFile(`${respBody.key}\n`,true);
	})
}

// 目录上传方法
async function uploadDirectory(dirPath) {
	fs.readdir(dirPath, function (err, files) {
		if (err) throw err;
		// 遍历目录下的内容
		files.forEach(item => {
			let path = `${dirPath}/${item}`;
			fs.stat(path, function (err, stats) {
				if (err) throw err;
				// 是目录就接着遍历 否则上传
				if (stats.isDirectory()) uploadDirectory(path);
				else {
					const key = path.replace(staticPath, prefix);
					// 删除相同环境下七牛云上与本地相同的文件
					if (key.includes(ENV)) {
						deleteSigleF(key).then(() => {
							uploadFile(path);
						}).catch(() => {
							uploadFile(path);
						})
					}
				}
			});
		})
	});
}

// 批量删除文件方法
function deleteFile(deleteOperations) {
	return new Promise((resolve, reject) => {
		//每个operations的数量不可以超过1000个，如果总数量超过1000，需要分批发送
		if (!deleteOperations) {
			console.log('删除的文件数组不存在');
			return false;
		}
		bucketManager.batch(deleteOperations, function (err, respBody, respInfo) {
			if (err) {
				console.log("文件批量删除出错");
				console.log(err);
			} else {
				// 200 is success, 298 is part success
				if (parseInt(respInfo.statusCode / 100) === 2) {
					respBody.forEach(function (item) {
						if (item.code === 200) {
							console.log("删除成功");
							// console.log(respBody);
							resolve("删除成功");
						} else {
							console.log("删除失败" + item.data.error);
							reject("删除失败" + item.data.error);
						}
					});
				} else {
					console.log(respInfo.deleteusCode);
					console.log(respBody);
					console.log('错误101--------------------');
					reject(respBody);
				}
			}
		});
	});
}

// 删除单个文件
function deleteSigleF(path) {
	return new Promise((resolve, reject) => {
		bucketManager.delete(bucket, path, function (err, respBody, respInfo) {
			if (err) {
				// console.log("删除失败" + err + "\t 路径:" + path);
				console.log("删除失败" + err);
				reject();
			} else {
				console.log(respBody, path);
				resolve();
			}
		});
	});
}

async function readFile(returnType = false) {
	return new Promise((resolve, reject) => {
		fs.readFile(`./file-${ENV}.txt`, {flag: 'r+', encoding: 'utf-8'}, function (err, data) {
			if (err) {
				reject(err);
			} else {
				let dataArr = data.split('\n');
				if (returnType) {
					resolve(dataArr);
					return;
				}
				console.log('\n上一次上传文件--------------------\n');
				console.log(dataArr);
				let deleteOperations = [];
				dataArr.forEach((item) => {
					if (item) {
						deleteOperations.push(qiniu.rs.deleteOp(bucket, item));
					}
				});
				resolve(deleteOperations);
			}
		});
	});
}


function baseFile(content, isAppend) {
	fs.writeFile(`./file-${ENV}.txt`, content, {flag: isAppend ? 'a' : 'w', encoding: 'utf-8', mode: '0666'}, function (err) {
		let str = "";
		if (isAppend) {
			str = (err ? "已上传但未记录:\t" : "已上传并记录:\t") + cdnPrefix + content.replace('\n', '');
		} else {
			str = err ? "文件写入失败" : ("删除服务器上相同文件,记录文件" + (content ? '变为' + content + '\n' : '清空---------------\n'));
		}
		console.log(str);
	});
}

function uploadHome() {
	// 清空文件内容
	baseFile('');
	fs.exists(staticPath, function (exists) {
		if (!exists) {
			console.log('目录不存在！');
		} else {
			uploadDirectory(staticPath);
			console.log('\n开始上传...');
		}
	})
}

// 1.读取本地保存的上一次上传记录
// 2.删除七牛云上一次上传记录的文件
// 3.删除七牛云上与本地相同的文件
// 4.上传文件并指定文件的MimeType
// 5.记录上传的文件
async function exec() {
	// let deleteOperations = await readFile();
	// console.log('\n删除上一次上传文件---------------\n');
	// if (deleteOperations.length) {
	//   // let deleteOP = await deleteFile(deleteOperations);
	//   deleteFile(deleteOperations).then((res) => {
	//     uploadHome();
	//   }).catch((err) => {
	//     uploadHome();
	//   })
	// } else {
	//   uploadHome();
	// }
	uploadHome();
}

exec();


```
运行 

`node qiniuUpload.js dev`
`node qiniuUpload.js prod`
`node qiniuUpload.js test`
