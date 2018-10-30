# UmengAndroidApk
Android借助友盟的多渠道打包使用方式

我们Android应用开发完成一般都是要放到不同的应用市场上供用户下载，运营人员在推广产品的时候要知道各个市场的下载数量方便针对不同市场的情况作出不同的
推广方案，这就要求我们开发人员给运营人员不同的apk包并标有每个市场唯一的渠道Id，就是多渠道打包，这次记录的是友盟的多渠道打包方式。

友盟我们都很了解了一般项目中都会使用到，其中最常用的就是统计这块的功能；
友盟实现多渠道打包的方法；

1、登录友盟官网 https://www.umeng.com  找到最上方的产品菜单，选择移动统计，进入之后找到他的文档介绍，先大概的浏览一遍整体的操作，知道这个东西是干嘛用的就行

2、注册友盟账号，进入个人中心找到我的产品，创建新的项目，友盟会提供给你一个唯一的appkey,这个Appkey在我们的项目配置中是要使用到的，之后就按照文档的提示一步一步的下载sdk或者使用远程依赖的方式添加友盟的资源，
配置manifest中的权限最重要的就是配置友盟的Appkey


<!--appkey 在友盟平台上创建应用获取到的-->
        <meta-data android:value="YOUR_APP_KEY" android:name="5bd7c27cb465f5d8590001d8"/>
        <!--友盟的渠道号，这里使用占位符的方式配置-->
        <meta-data android:value="Channel ID" android:name="${CHANNEL_ID_VALUE}"/>

       这里面的channelID也可以直接写 上要打的渠道的名称，但是比较麻烦所以使用gradle的方式配置就可以一次把所有的渠道包都打出来了，更加方便

3、开始配置gradle,我们创建一个项目的时候会有三个gradle文件，第一个settings.gradle主要是配置model的，如果项目中有多个model都会这这里体现；第二个根目录下的build.gradle,这个文件主要是配置我们的项目gradle的版本，还有远程的仓库，比如jcenter，maven等一些依赖；第三个就是app目录下的build.gradle文件，这个文件
就是我们要配置多渠道打包的地方了，这里主要配置项目的SDK版本，应用版本，三方库的依赖等；

gradle的配置方法：
1) 先在defaultConfig下配置multiDexEnabled true解决方法数量65535的限制问题
设置我们manifest中占位符CHANNEL_ID_VALUE的默认值 manifestPlaceholders = [CHANNEL_ID_VALUE: "umeng"]

        defaultConfig {
            applicationId "tanganna.com.umengandroidapk"
            minSdkVersion 15
            targetSdkVersion 28
            versionCode 1
            versionName "1.0"
            testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
            multiDexEnabled true  //突破应用方法数量65535的限制
            manifestPlaceholders = [CHANNEL_ID_VALUE: "umeng"]  //设置默认的友盟渠道号

        }


2) 在buildTypes下配置签名文件( 生成签名文件的方式：在build菜单中选择Generate signed bundle -->选择apk -->选择签名文件要存储的路径，设置密码、别名、别名密码、下面的填写至少一项就可以)
主要是配置签名文件的路径、密码、别名、别名密码一般只配置release就可以，debug空着就行

        buildTypes {
                release {
                    minifyEnabled false
                    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

                    signingConfig signingConfigs.release
                }
        }

接着就要写signingConfigs

        //添加签名文件的配置
            signingConfigs {
                debug {}

                release {
                    storeFile file("tanganna.jks")  //签名文件路径
                    storePassword "tan123456"       //密码
                    keyAlias "UmengApk"             //别名
                    keyPassword "tan123456"         //别名密码
                }
        }

3) 配置渠道号使用productFlavors

        //配置渠道号
            flavorDimensions "default"
            productFlavors {

                xiaomi {
                    manifestPlaceholders = [CHANNEL_ID_VALUE: "xiaomi"]
                }

                wandoujia {
                    manifestPlaceholders = [CHANNEL_ID_VALUE: "wandoujia"]
                }

                yingyongbao {
                    manifestPlaceholders = [CHANNEL_ID_VALUE: "yingyongbao"]
                }

        //        .....其他渠道配置
            }

简单方式配置渠道号通过循环的方式直接使用渠道名设置渠道号

        //简单的配置渠道的方式
            productFlavors.all {
                flavor -> flavor.manifestPlaceholders = [CHANNEL_ID_VALUE: name]
            }

此时就已经实现了多渠道打包的代码，进行打包操作就可以打出所有的渠道包，将所有的渠道包交给运营人员就可以了；


这是生成的apk的名称都差不多看起来不是很好的区分，我们还可以配置生成的apk的名称

4) 还是在buildTypes下继续写脚本

        buildTypes {
                release {
                    minifyEnabled false
                    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

                    signingConfig signingConfigs.release

                    //指定我们的输出的文件的名就是我们的渠道名
                    android.applicationVariants.all{variant ->
                        variant.outputs.all{
                            output ->
                                def outFile = output.outputFile
                                if(outFile != null && outFile.name.endsWith(".apk")){
                                    def fileName = "${variant.productFlavors[0].name}" + ".apk"
                                    outputFileName=fileName;
                                }
                        }
                    }
                }
            }

这时生成的apk都是以渠道名命名的了比较好区分




注意：
1) AndroidStudio不同版本中写gradle脚本有所不同需要注意
2) 占位符替换时一定要两边写的一致