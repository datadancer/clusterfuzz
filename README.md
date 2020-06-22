# 使用最新的clusterfuzz和需要解决的问题

## 建立环境
* 下载clusterfuzz_master.tar.gz
* 解压gzip -d clusterfuzz_master.tar.gz得到clusterfuzz_master.tar
* 加载docker load -i clusterfuzz_master.tar，加载成功后会得到镜像22ed944b6490
* 启动docker run -it --name clusterfuzz --network host 22ed944b6490 /bin/bash
* 进入docker后，执行以下指令用于启动clusterfuzz
   ```su - cf
   cd /clusterfuzz
   . "$(python3.8 -m pipenv --venv)/bin/activate"
   python butler.py run_server -b --skip-install-deps
   ```
   执行示例如下：
```
$ docker run -it --rm --name clusterfuzz --network host 22ed944b6490 /bin/bash
root@YT:/# su - cf
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
## 错误重现
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

* 设置好点击ADD得到文件上传错误
```
Failed to upload. (test@example.com)   File "/usr/lib/python2.7/threading.py", line 774, in __bootstrap
    self.__bootstrap_inner()
  File "/usr/lib/python2.7/threading.py", line 801, in __bootstrap_inner
    self.run()
  File "/usr/lib/python2.7/threading.py", line 754, in run
    self.__target(*self.__args, **self.__kwargs)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/tools/devappserver2/thread_executor.py", line 41, in _worker
    result = fn(*args, **kwargs)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/tools/devappserver2/wsgi_server.py", line 116, in _handle
    obj.communicate()
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/cherrypy/cherrypy/wsgiserver/wsgiserver2.py", line 1302, in communicate
    req.respond()
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/cherrypy/cherrypy/wsgiserver/wsgiserver2.py", line 831, in respond
    self.server.gateway(self).respond()
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/cherrypy/cherrypy/wsgiserver/wsgiserver2.py", line 2115, in respond
    response = self.req.server.wsgi_app(self.env, self.start_response)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/tools/devappserver2/wsgi_server.py", line 292, in __call__
    return app(environ, start_response)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/tools/devappserver2/request_rewriter.py", line 314, in _rewriter_middleware
    response_body = iter(application(environ, wrapped_start_response))
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/tools/devappserver2/python/runtime/request_handler.py", line 160, in __call__
    response = self.handle_normal_request(environ)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/tools/devappserver2/python/runtime/request_handler.py", line 195, in handle_normal_request
    self._PYTHON_LIB_DIR)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/runtime/runtime.py", line 159, in HandleRequest
    error)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/runtime/wsgi.py", line 329, in HandleRequest
    return WsgiRequest(environ, handler_name, url, post_data, error).Handle()
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/google/appengine/runtime/wsgi.py", line 267, in Handle
    result = handler(dict(self._environ), self._StartResponse)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/webapp2-2.3/webapp2.py", line 1505, in __call__
    rv = self.router.dispatch(request, response)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/webapp2-2.3/webapp2.py", line 1253, in default_dispatcher
    return route.handler_adapter(request, response)
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/webapp2-2.3/webapp2.py", line 1077, in __call__
    return handler.dispatch()
  File "/usr/lib/google-cloud-sdk/platform/google_appengine/lib/webapp2-2.3/webapp2.py", line 545, in dispatch
    return method(*args, **kwargs)
  File "/clusterfuzz/src/appengine/libs/handler.py", line 286, in wrapper
    return func(self, *args, **kwargs)
  File "/clusterfuzz/src/appengine/libs/handler.py", line 413, in wrapper
    return func(self, *args, **kwargs)
  File "/clusterfuzz/src/appengine/handlers/jobs.py", line 126, in post
    blob_info = self.get_upload()
  File "/clusterfuzz/src/appengine/handlers/base_handler.py", line 248, in get_upload
    raise helpers.EarlyExitException('Failed to upload.', 500)
  File "/clusterfuzz/src/appengine/libs/helpers.py", line 48, in __init__
    self.trace_dump = ''.join(traceback.format_stack())

```
   这个文件上传错误问题在哪里？怎么解决？

