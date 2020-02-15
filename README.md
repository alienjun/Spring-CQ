# Spring-CQ
[![Build Status](https://travis-ci.org/lz1998/spring-cq.png)](https://travis-ci.org/lz1998/spring-cq)
[![QQ群](https://img.shields.io/static/v1?label=QQ%E7%BE%A4&message=335783090&color=blue)](https://jq.qq.com/?_wv=1027&k=5BKAROL)

基于 酷Q、cqhttp、SpringBoot、反向websocket 的 QQ 机器人框架

## 编写插件
1. 在xin.lz1998.bot.plugin下写XXXPlugin，继承CQPlugin  
    ```java
    /**
     * 示例插件
     * 插件必须继承CQPlugin
     *
     * 添加事件：光标移动到类中，按 Ctrl+O 添加事件(讨论组消息、加群请求、加好友请求等)
     * 查看API参数类型：光标移动到方法括号中按Ctrl+P
     * 查看API说明：光标移动到方法括号中按Ctrl+Q
     */
    @Slf4j
    public class ExamplePlugin extends CQPlugin {
        /**
         * 收到私聊消息时会调用这个方法
         *
         * @param cq    机器人对象，用于调用API，例如发送私聊消息 sendPrivateMsg
         * @param event 事件对象，用于获取消息内容、群号、发送者QQ等
         * @return 是否继续调用下一个插件，IGNORE表示继续，BLOCK表示不继续
         */
        @Override
        public int onPrivateMessage(CoolQ cq, CQPrivateMessageEvent event) {
            // 获取 发送者QQ 和 消息内容
            long userId = event.getUserId();
            String msg = event.getMessage();
    
            if (msg.equals("hi")) {
                // 调用API发送hello
                cq.sendPrivateMsg(userId, "hello", false);
    
                // 不执行下一个插件
                return MESSAGE_BLOCK;
            }
            // 继续执行下一个插件
            return MESSAGE_IGNORE;
        }
    
     
        /**
         * 收到群消息时会调用这个方法
         *
         * @param cq    机器人对象，用于调用API，例如发送群消息 sendGroupMsg
         * @param event 事件对象，用于获取消息内容、群号、发送者QQ等
         * @return 是否继续调用下一个插件，IGNORE表示继续，BLOCK表示不继续
         */
        @Override
        public int onGroupMessage(CoolQ cq, CQGroupMessageEvent event) {
            // 获取 消息内容 群号 发送者QQ
            String msg = event.getMessage();
            long groupId = event.getGroupId();
            long userId = event.getUserId();
    
            if (msg.equals("hello")) {
                // 回复内容为 at发送者 + hi
                String result = CQCode.at(userId) + "hi";
    
                // 调用API发送消息
                cq.sendGroupMsg(groupId, result, false);
    
                // 不执行下一个插件
                return MESSAGE_BLOCK;
            }
    
            // 继续执行下一个插件
            return MESSAGE_BLOCK;
        }
    }
    ```
    私聊截图
    ![私聊截图](http://cq.lz1998.xin/screenshot_private.png)
    
    群聊截图
    ![群聊截图](http://cq.lz1998.xin/screenshot_group.png)

2. 修改PluginConfig，配置顺序
    ```java
    /**
     * 插件配置类
     * 在pluginList中配置需要执行插件的顺序
     * 收到消息后会按顺序调用插件
     * <p>
     * 提示：
     * 如果前一个插件返回MESSAGE_BLOCK，那么之后的插件不会继续处理
     * 如果前一个插件返回MESSAGE_IGNORE，那么之后的插件会继续处理
     */
    public class PluginConfig {
        public static List<CQPlugin> pluginList = new ArrayList<>();
    
        static {
            pluginList.add(new StatusPlugin()); // 状态监控插件
            pluginList.add(new LogPlugin()); // 日志插件
            // pluginList.add(new PrefixPlugin()); //前缀处理插件 如果需要给所有指令加上前缀，比如“.”、“/”，可以使用这个插件在此统一处理
            pluginList.add(new ExamplePlugin()); // 示例插件
        }
    }
    ```



    
## 测试应用
1. 修改application.properties的端口，默认8081
2. 运行SpringCqApplication的main方法

## 打包应用
1. 使用maven打包应用
    ```bash
    mvn clean package
    ```
2. 在target目录下，spring-cq-0.0.1-SNAPSHOT.jar即为打包的jar

## 运行应用
1. 输入指令
    ```bash
    java -jar spring-cq-0.0.1-SNAPSHOT.jar
    ```
如果是Windows，并且不需要查看运行情况，可以直接双击jar文件运行，右下角托盘会出现小图标

## Windows运行酷Q和cqhttp
1. 准备酷Q Air
    - 方案一：下载已经配置好cqhttp的[酷Q Air](http://cq.lz1998.xin/CQA.zip)
    - 方案二：自己配置
        1. 下载[酷Q Air](https://cqp.cc/t/23253)
        2. 下载[CQHTTP插件](https://github.com/richardchien/coolq-http-api/releases)
        3. 创建文件`酷Q Air\data\app\io.github.richardchien.coolqhttpapi\config.ini`
            ```ini
            [general]
            use_http=false
            use_ws_reverse=true
            ws_reverse_url=ws://127.0.0.1:8081/ws/cq/
            ws_reverse_use_universal_client=true
            enable_heartbeat=true
            heartbeat_interval=60000
            ```
2. 解压后运行 CQA.exe 登录QQ账号 




## Docker运行酷Q和cqhttp
1. 安装酷Q和CQHTTP插件
    ```shell
    docker run -d --name cq01 \
    -v $(pwd)/coolq:/home/user/coolq \
    -p 9000:9000 \
    -e VNC_PASSWD=你的VNC密码(不超过8位) \
    -e COOLQ_URL=http://dlsec.cqp.me/cqa-tuling \
    -e COOLQ_ACCOUNT=你的机器人QQ号 \
    -e CQHTTP_USE_HTTP=false \
    -e CQHTTP_USE_WS_REVERSE=true \
    -e CQHTTP_WS_REVERSE_URL=ws://宿主机地址:8081/ws/cq/ \
    -e CQHTTP_WS_REVERSE_UNIVERSAL_CLIENT=true \
    -e CQHTTP_ENABLE_HEARTBEAT=true \
    -e CQHTTP_HEARTBEAT_INTERVAL=60000 \
    richardchien/cqhttp
    ```
    如果不知道宿主机地址是什么，可以使用Docker的host模式，共享主机网络
    ```bash
    docker run -d --name cq01 \
    -v $(pwd)/coolq:/home/user/coolq \
    --net=host \
    -e VNC_PASSWD=你的VNC密码(不超过8位) \
    -e COOLQ_URL=http://dlsec.cqp.me/cqa-tuling \
    -e COOLQ_ACCOUNT=你的机器人QQ号 \
    -e CQHTTP_USE_HTTP=false \
    -e CQHTTP_USE_WS_REVERSE=true \
    -e CQHTTP_WS_REVERSE_URL=ws://127.0.0.1:8081/ws/cq/ \
    -e CQHTTP_WS_REVERSE_UNIVERSAL_CLIENT=true \
    -e CQHTTP_ENABLE_HEARTBEAT=true \
    -e CQHTTP_HEARTBEAT_INTERVAL=60000 \
    richardchien/cqhttp
    ```
2. 访问 http://127.0.0.1:9000 登录QQ账号
