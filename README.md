# 使用最新的clusterfuzz和需要解决的问题

## 建立环境
* 下载clusterfuzz_master.tar.gz
* 解压gzip -d clusterfuzz_master.tar.gz得到clusterfuzz_master.tar
* 加载docker load -i clusterfuzz_master.tar，加载成功后会得到镜像22ed944b6490
* 启动docker run -it --name clusterfuzz --network host 22ed944b6490 /bin/bash
* 进入docker后，执行以下指令用于启动clusterfuzz
   ```su - cf
   export GOROOT=/usr/local/go
   export GOPATH=/clusterfuzz
   export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
   cd /clusterfuzz
   . "$(python3.8 -m pipenv --venv)/bin/activate"
   python butler.py run_server -b --skip-install-deps
   ```
   执行示例如下：
```
$ docker run -it --rm --name clusterfuzz --network host 22ed944b6490 /bin/bash
root@YT:/# su - cf
$ export GOROOT=/usr/local/go
$ export GOPATH=/clusterfuzz
$ export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
$ cd /clusterfuzz
$ . "$(python3.8 -m pipenv --venv)/bin/activate"
(clusterfuzz) $ python butler.py run_server -b --skip-install-deps
Running: pkill -KILL -f "dev_appserver.py"
| Return code is non-zero (-9).
Running: pkill -KILL -f "CloudDatastore.jar"
| Return code is non-zero (-9).
Running: pkill -KILL -f "pubsub-emulator"
| Return code is non-zero (-9).
Running: pkill -KILL -f "run_bot"
| Return code is non-zero (-9).
Created symlink: source: /clusterfuzz/configs/test, target /clusterfuzz/src/appengine/config.
Created symlink: source: /clusterfuzz/src/protos, target /clusterfuzz/src/appengine/protos.
Created symlink: source: /clusterfuzz/src/python, target /clusterfuzz/src/appengine/python.
Running: python polymer_bundler.py (cwd='local')
| App Engine templates are up to date.
Clearing local datastore by removing local/storage.
Created symlink: source: /clusterfuzz/local/storage/local_gcs, target /clusterfuzz/src/appengine/local_gcs.
Running: gunicorn -b :9000 main:app (cwd='src/appengine')
| [2020-06-22 02:07:06 +0000] [240] [INFO] Starting gunicorn 20.0.4
| [2020-06-22 02:07:06 +0000] [240] [INFO] Listening at: http://0.0.0.0:9000 (240)
| [2020-06-22 02:07:06 +0000] [240] [INFO] Using worker: sync
| [2020-06-22 02:07:06 +0000] [244] [INFO] Booting worker with pid: 244
Bootstrapping datastore...
Running: python butler.py run setup --non-dry-run --local --config-dir=configs/test
| Creating config
| Creating fuzzer afl
| Creating fuzzer libFuzzer
| Creating fuzzer honggfuzz
| Creating fuzzer syzkaller
| Creating template afl
| Creating template engine_asan
| Creating template engine_msan
| Creating template engine_ubsan
| Creating template honggfuzz
| Creating template libfuzzer
| Creating template syzkaller
| Creating template prune
| Done
```
## 添加Job
* 启动成功后用浏览器打开http://0.0.0.0:9000进入clusterfuzz的使用界面
* 点击左边菜单栏的Jobs进入到http://127.0.0.1:9000/jobs页面，找到Add new job
   并按照如下设置上传文件，参考页面https://testerhome.com/topics/18171
```   
   输入框名称	内容
   Name		libfuzzer_asan_linux_openssl
   Platform	LINUX
   Templates	libfuzzer engine_asan
   Environment 	CORPUS_PRUNE = True
```

* 设置好点击ADD得到文件上传成功，做一些使用的测试

