# flutterw
flutter hybrid project cli

目前仅支持iOS



## How to use

1.在iOS或安卓项目的根目录中创建一个flutter工程目录，例如可命名为FlutterProject

```shell
mkdir -p /ProjectRoot/FlutterProject
```



2.在git上创建一个空工程，作为flutter工程，然后clone到Flutter工程目录

```shell
cd /ProjectRoot/FlutterProject
git clone $YourFlutterProject
```



3.在原生工程中将flutter工程忽略：在iOS或安卓主工程的.gitignore文件中添加 

```c
FlutterProject/$YourFlutterProjectName
```



4.创建flutter module

```shell
flutter create -t module $YourFlutterProjectName #名字就是在git上创建的工程名
```



5.将flutterw文件拷贝到flutter工程的根目录



6.执行flutterw初始化flutter项目

```shell
chmod +x ./flutterw
./flutter
```



7.打包

```shell
./flutterw build module ios
./flutterw build module android
```



.iOS中集成

a.按照官方文档集成：https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps

**按照a步骤走完再将b，c步骤替换即可**

b.修改Podfile (flutter_module_demo)

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
use_frameworks!

def addFlutterProject
  
  isFlutterEnable = false
  flutterProjectName = '$YourFlutterProject'
  flutter_application_path = 'FlutterProject/' + flutterProjectName + '/'
  flutterConfigPath = 'project.properties'
  if File::file?(flutter_application_path + flutterConfigPath)
    IO.foreach(flutter_application_path + flutterConfigPath) { | line |
      if line == "FlutterEnable=true\n"
        isFlutterEnable = true
        break
      end
    }
  end
  
  if isFlutterEnable
    puts 'Flutter mode'
    
    eval(File.read(File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')), binding)
    
  else
    puts 'Native mode'
    
    pod 'Flutter',                      :path => 'FlutterProduct/Flutter'
    pod flutterProjectName,             :path => 'FlutterProduct/' + flutterProjectName
    pod 'FlutterPluginRegistrant',      :path => 'FlutterProduct/FlutterPluginRegistrant'
    pod flutterProjectName + '_plugins',  :path => 'FlutterProduct/' + flutterProjectName + '_plugins'

  end
  
end

target '$iOSProjectName' do
  
  addFlutterProject()
  
end
```

c.修改Project -- Target -- Build Phases中的Flutter脚本

```shell
isFlutterEnable() {
  local flutterProjectName="flutter_module_demo"
  local flutterConfigPath="FlutterProject/$flutterProjectName/project.properties"
  if [[ ! -e $flutterConfigPath ]]; then
    return 0
  fi

  local isEnable=$(grep -c "FlutterEnable=true" $flutterConfigPath)
  if [ $isEnable -eq '1' ]; then
    return 1
  fi

  return 0
}

isFlutterEnable
if [ $? -eq 1 ]; then
  "$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" build
  "$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" embed
fi
```



9.flutter开发模式开关

```shell
./flutter enable true		#打开flutter开发模式
./flutter enable false	#关闭flutter开发模式
```

