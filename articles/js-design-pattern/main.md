# Js 中的14种设计模式

> GoF 四人组总结出23种设计模式,后人对设计模式的评价总体来说赞誉多一点,也有负面意见,认为设计模式就是语言这件衣服的补丁。JavaScript 这件衣服不算太破,前辈们总结出了14种模式来弥补语言设计的"缺陷",在很多事情上选择是对立的,所以这里的缺陷只是一种选择。其实设计模式另一个来源就是实践中总结出来的套路,比如适配器模式在平滑迁移老代码这种场景下特别好用。这篇文章是《JavaScript 设计模式与开发实践》的读书笔记，共勉。

## 单例模式

定义: 保证一个类只有一个实例,并提供一个访问它的全局访问点。

对于静态语言(JAVA,C++等)需要对象需要先创建类，而 JS 可以直接定义一个对象。

单例模式主要是两个应用场景：
- 提供全局的工具方法(自执行函数和命名空间对外暴露函数列表)
- 页面中零到多次的功能模块(登录系统错误警告)

这是一个生成序列 id 的函数，通过闭包实现这样一个公共函数，这个函数不管用不用都在那里，有点就是调用方便，缺点是如果一次都没有使用过会浪费内存：

    var createId = (function () {
        var id = 1;
        return function () {
            return id++;
        }
    })();    

还有一种单例是用的时候每次都生成一次，但是在原理上复用同一个内存对象，这个需要用到工厂模式，优点是节省内存，由于 js 是解释型语言如果没用使用过对象是不会在内存中存在的，缺点就是每次使用都需要生成一个：
    
    var ObjectAFactory = (function () {
        var t;
        return function () {
            // 如果第一次调用就生成一个对象，并利用作用域特性保存起来
            // 否则这届放回第一次生成的对象
            if (!t) {
                t = {
                    description: '这可能是个很大很复杂的对象'
                };
            }
            return t;
        };
    })();

    // 每次使用时都要生成一下，工厂会判断是否已经存在过
    var singleObject = ObjectAFactory();

单例不管怎么实现还有一个共同的缺点，就是在一处修改了单例后影响到后面程序的使用，所以要严谨修改单例。
 
## 策略模式

定义：定义一系列方法，使它们之间可以相互替换。

将变化的部分和不变的部分隔离开是包括策略模式在内的多个设计模式的目的。策略模式面向使用者的优点在于封装了不同输入条件调用不同的策略，对策略本身的开发优势在于策略之间的相互隔离；策略模式的局限性在于各个策略之间的排他性设计，如果要添加或修改某个策略需要了解整个计算规则以免在逻辑上有叠加或断层，这时对策略的注释就尤为重要。

策略的适用场景不多，要求根据参数可以抽象出一个模式来，但如果模式逻辑很简单似乎将策略抽象为一个配置文件，比如算工资的那个示例可以做下面改写：

    function calculateBonus(performanceLevel, salary) {
        var config = {
            A: 3,
            B: 2,
            S: 4
        };
        // 这里省去了对参数的校验...
        
        return config[performanceLevel] * salary;
    }    

代码的简洁性一目了然，如果在这样的计算逻辑下添加或修改一个策略成本比示例的方式要低得多。但是如果新策略不是 `performanceLevel*salary` 这样的计算逻辑那这个代码是不是就尴尬了？

我们说两条：
- 程序的扩展性和过度设计之间的界限要把握好；
- 策略逻辑要尽量简化。

我们不知道级别 C 和新的计算逻辑谁先出现，或许他们都不会出现，所以在不知道新需求的方式时当前够用的就是最好的，如果有新的计算逻辑可以这样改造上面函数：
 
    function calculateBonus(performanceLevel, salary) {
        var config = {
            A: handler: function () {
                return 3 * salary;
            }
        };

        var item = config[performanceLevel];
        // 这里省去了对参数的校验...
        
        return item.handler();
    }

没有最好的代码，只有最适合的代码。用配置的方式组织策略，简单到可能认不出这是策略模式。在策略模式中根据策略的异同点来抽象策略的组织是此模式的关键。

## 代理模式

定义：为一个对象提供一个代用品或占位符，以便控制对他的访问。

代理的主要目的是增强原有对象，增强的方式是用户通过访问代理对象，代理对象为被代理对象实现一部分功能。代理有三个比较好的应用场景：

- 为特定场景定制原对象，使调用更简单(将原对象的一些接口固化)；
- 将某些场景的通用逻辑封装到代理中，减少重复代码；
- 使依赖反转，方便偷梁换柱(这在切换库函数时尤为有效)

## 迭代器模式(略)

在 ES5 中此模式已经有了很好的支持，如数组原生支持 `forEach` `reduce` 等常规迭代模式，扩展的还有 `some` `every` `reduceRight` `map` `filter` 这些，所以此模式在 js 中可以略去。

## 发布-订阅模式(观察者模式)

定义：它定义对象间的一种一对多的依赖关系，当一个对象的状态发生变化时，所有依赖它的其他对象都将得到通知。

此模式有三个层级的实现：

一、模拟 Dom 事件，发布者全局唯一(浏览器宿主进行事件派发)，每一个订阅者自带订阅方法，经典代码如下：

    document.getElementById('id').addEventListener('click', function () {
        // 响应逻辑
    });

二、全局模式，订阅者和发布者共同组成一个单例，当然这个单例需要出现在订阅和发布之前，作用域下都可以订阅事件和发布事件，好处在于方便，但坏处是事件可以满天飞，需要解决命名冲突的问题，书中给出的 `create('namespace')` 方案不是很优雅，这里斗胆提另一种方案：

    Event.listen({
        scope: 'namespace',
        name: 'click',
        handler: function () {
            // 响应逻辑，也可以把此函数放在对象后作为单独的参数
        }
    });

通过第一个参数类型来判断需不需要走命名空间。这种全局的发布-订阅模式在早期的粗放式开发中可能会用到，但是在今天到处模块化的今天还是用下面的模式比较好。

三、这种是前两种的结合类型，在组件中提供冒泡和广播机制，页面由组件组成，最上面有一个根组件(类似于 Dom 结构的 body)，下面组件通过符合排列等方式组成页面，事件的发布只能向上或向下，这样在一条组件链上天然形成了作用域分割。这种模式有一个缺点，当相邻组件需要通过发布-订阅通信时需要借助他们共同的父组件才能实现，Vue 1.x 就是用的此种模式。

从解耦的角度来看第一种方法使发布者不依赖订阅者，第二种方法使发布者和订阅者相互不依赖但但产生了作用域的问题，第三种方式通过 Dom 树的天然分支特性来分割作用域。

## 命令模式(略)

在 js 中这个模式已经融入到语法特性中，可用回调更方便的实现此模式的目标。

## 组合模式

在实现上 js 中一种具有天然优势的设计模式，对外忽略部分和整体的区别，提供一种"用一个调用处理一串对象"的方法。比如大家都熟悉的 jQuery 操作 Dom 的模式：

    $('.handler').click(function () {      
        // ...
    });
    
一个 jQuery 对象就是将一组 Dom 节点包装成一个对象，然后通过此对象操作这一组 Dom 而不用管这组 Dom 具体是什么标签、有多少个 Dom 节点组成等细节。项目中的组合模式可能是多层结构，我们需要一个功能模块来统一管理这种结构，也需要每个模块暴露统一的对外接口来被管理，二者缺一不可并规则要明确。

在一个店铺多模版装修系统中我们有过组合模式的实践，每个组件的输入和输出参数要符合一定的规则才能接入进模板系统中，说几个最基本的参数：

    filed      // 数据输出和数据回写的必要参数
    validate   // 验证函数，管理器在获取数据时必然会调用的方法
    getValue   // 获取数据函数
    setValue   // 回写数据函数

只要保证每个组件在组合对象中上下行为一致，配置和操作起来都会变得简单明了起来。

组合对象有一个缺点，组合中的对象不管是最上层中间层还是最底层在涉及到数据上行下行和对象组织方式等关键逻辑上都不能有特殊化，如果有那就不适合用此模式，如果已经用了此模式但是业务需求出现了特例(当初设计的逻辑覆盖不到当前的业务需求)，那就需要再添加逻辑的复杂性使特例变成通例，或者将特例进行封装(在特例中可能要有一些冗余重复的代码)。

这里有一个特例，当初需要这样一个数据结构的表单：

    [
        {
            title: '60平以下装修方案',
            price: 80000, // 报价(元)
            time: 30      // 工期(天) 
        },
        {
            title: '60-100平装修方案',
            price: 150000,
            time: 40
        },
        {
            title: '100平以上装修方案',
            price: 200000,
            time: 40
        }
    ]

但是三个数据对象不是全必须，也就是可能出现数组中只有两条或一条数据的情况，提交数据没有问题，但是回写的时候就出现问题了，当给了两条数据那我要回写到1、2还是2、3还是1、3，当初设计的管理器是处理不了这种需求的，那面对这种情况我们要采用哪种方案呢，是添加管理器的复杂度还是将这个特例封装？

我们使用了后一种解决方案，封装了一个单独的模块来满足这个需求。为什么用后一种，或者说如何决定用哪一种？如果现在这个需求是特定场合产生的，在可预知的需求中没有类似需求，就比如当前的这个需求是家装模板的装修方案的报价和排期场景下的需求，在可预知的行业模板中想不到类似的需求，所以我们选了第二种方案。当然这里使用第一种方案也能解决问题，但是从开发成本和后期的维护成本来看都不合适，还有过度设计的嫌疑。

## 模板方法模式

要解决的问题：两个对象有一个方法(我们称为"模板方法")，它调用了对象的其他方法，对两个对象而言其中有相同的有不同的，此模式就是要消灭相同方法在两个对象中的重复代码，分离两个对象的不同代码。

解决方法：用抽象类抽取模板方法中调用的公共方法，子类覆盖模板方法中调用的非公共方法。

此模式的适用范围比较窄，局限性主要集中在模板方法中，其实模板方法把两个对象的行为都做了限定，两个对象相互牵制，不太好适应需求的变化。

好莱坞原则 和 非继承实现模板方法模式是很好的两个例子。

## 享元模式

主要作用是节约内存，没有内部状态就成了单例工厂，没有外部状态就成了对象池。

享元和对象池都要用到一个小技巧，利用闭包特性用工厂的方式缓存对象，这一技巧在单例中同样适用，代码如下：

    var ObjectAFactory = (function () {
        var t;
        return {
            create: function () {
                if (!t) {
                    t = new ObjectA();
                }
                return t;
            }
        }
    })();
    
    // 每次使用时这样来声明，如果一次都没用到就剩内存了
    // 如果多次用到在内存中也只有一份
    var a = ObjectAFactory.create();

这段是 `ObjectA` 这一对象的一个工厂，对内部状态的初始化可能需要还需要策略模式的辅助，在内衣的例子中看着比较傻(明明可以用参数做的事情非要声明连个对象)，但在复杂一点的例子(如文件上传)中内部状态需要一个初始化的过程才能对外提供一致的可用方法，所以如果不需要这个初始化过程的话，也就是说没有内部状态的差异的话，这个享元模式就成为了工厂单例。

早期的 IE(<9) 中的 `event` 就是享元模式，在事件中如果想要拿到当前的事件对象直接使用 `window.event` 就可以(不需要显示的在回调函数中声明事件对象参数)。使用享元模式有一个缺陷需要用其他方式弥补，内部状态肯定只有一份，外部状态特性如果不是队列(数组)后面的赋值会覆盖前面的赋值，如果特性是队列并多处使用享元对象要做好辅助隔离工作。

后面的赋值会覆盖前面的赋值代码示例：

    var a = ObjectAFactory.create();
    var b = ObjectAFactory.create();
    
    a.attr = 'a';
    b.attr = 'b';
    // 这时 a.attr 已经变成 'b' 了，这是所谓的后面的覆盖前面的

隔离工作示例

    a.arrayAttr.push('a');
    b.arrayAttr.push('b');
    // 这时 a.arrayAttr 和 b.arrayAttr 的值已经成为了 ['a', 'b']
    // 这是我们可能需要这样：
    a.arrayAttr.push({
        scope: a,
        value: 'a'
    });
   
这在输入和输出队列元素上都会显得比较怪异，也是享元模式不适用的场景 --- 需要反复使用享元对象但是每个享元对象的状态需要被区分不可被覆盖。
    
## 职责链模式

定义：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之前的耦合关系，这些对象形成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

链在数据结构中是一种和栈与队列相似的数据结构，链和队列的不同之处在于：链是元素之间自行指定向下级关系的(如双向链和二叉树)，而队列是靠数组把元素串起来，元素之间没有指向关系。所以职责链应该还有个亲兄弟叫"职责队列"，书中的预约销售和上传文件应该使用"职责队列"更为适合，链对于队列的优势在于如果要插入或删除其中的一个元素需要修改上一个并知道下一个，优势在于每个节点可以自行控制下一个是哪一个以及是否到下一个，如果判断逻辑相同那么每个节点中又必然会存在判断逻辑。如果你感兴趣可以用职责队列的方法改造书中的两个例子。下面是后台服务器框架中处理路由的一个例子：

场景描述：当客户端的一个请求打到服务器上时我们要请求进行处理，这个请求是静态文件还是数据接口请求，请求可以直接对应到静态文件或接口，但是也可以通过配置来控制请求路径和后台处理逻辑的对应关系，这就是服务器路由的常规场景，如果我们使用现成的服务器框架不会考虑到这些事情，但如果我们要定制或根据自己的具体项目优化服务器框架时这些东西就有必要了解了：

关键代码：

    function routRequest(request, response) {
        // 路由队列（路由的优先顺序在此配置）
        var routList = [
            routUserSettingPath,
            routStaticFile,
            routAutoPath,
            response404
        ];
    
        var rout = {
            next: function () {
                if (routList.length > 0) {
                    routList.shift()(request, response, rout);
                }
            },
            last: function () {
                // 拼接额外的参数
                var arg = [request, response].concat([].slice.call(arguments));
                routList.pop().apply(null, arg);
            }
        };
        // 错误日志，异常处理
        try {
            rout.next();
        }
        catch (e) {
            console.log('静态文件路由错误: ');
            console.log(e);
        }
    }

    function routUserSettingPath(request, response, rout) {
        // 代码省略
    }
    function routStaticFile(request, response, rout) {
        // 请求路径
        var path = url.parse(request.url).pathname;
    
        // 取得后缀名作为文件类型
        var contentType = path.match(/(\.[^.]+|)$/)[0];
        var staticFieldConfig = getStaticFieldConfig(contentType);
    
        // 允许访问此扩展名的静态文件
        if (staticFieldConfig !== undefined) {
            // 读取静态文件
            fs.readFile(config.webRootPath + path, staticFieldConfig.encoding, function (err, data) {
                if (err) {
                    rout.last({
                        type: 'file',
                        data: staticFieldConfig,
                        msg: '未找到文件，或文件读取失败'
                    });
                }
                else {
                    response200(response, contentType, data, staticFieldConfig.encoding);
                }
            });
        }
        // 不允许访问此扩展名的静态文件
        else {
            rout.next();
        }
    }

每个处理环节不关心下一个是哪一个环节，这样每个处理环节之间的依赖和耦合就解除了，变成了依赖抽象 `next` 和 `last`，当前没有处理成功就调用 `next`，如果处理成功就调用 `last`，并且在 `last` 函数中 还可以处理一些公共的逻辑，每个环节只关心自己最核心的处理。如果要添加或删除某一环节只要在在数组中添加或删除一个元素就可以。

## 中介者模式

它的作用是解除对象与对象之间的耦合关系，其实具体的讲这种关系应该是独立于各个对象但又和各个对象有关系的一种关系，比如书中的泡泡游戏的例子中一个队失败了，失败这个关系不能由任何一个队友单独决定但又和每个队友有关系，所以我们需要一个中心对象来处理这种关系，这样每个队员都不用处理这层关系，这层关系的处理者就成为了"中介"。

此模式的有点和缺点都在"中介者"这一对象上，如果中介者承担了太多的逻辑那必然会变得难以维护，在设计上每个中介者的一个方法最多只承担一种关系的处理(也可能是一种关系的部分处理)，不同关系之间的相互影响通过同一中介者的方法调用来处理，如果遇到异常复杂的关系处理可能需要拆分中介者，可能出现中介者的中介者。也可以重新考虑对象之间的关系是否合理，能不能换个维度来描述关系，使关系不需要那么多中介者就能很好的处理。

一个更常见的例子就是表格中的"全选/全不选/反选"的例子，是中介者模式很适用的场景。

## 装饰者模式(略)

不赞成动 `Function`，如果用包装的方式就是挂羊头卖狗肉了，还不如直接用代理模式再起个名字，因此此模式略过。

https://zhuanlan.zhihu.com/p/20743493

## 状态模式

状态实际成效的时机可以放在切换完成前也可以放在切换完成后，用 `if-else` 或 `switch` 简单实现是典型的状态切换完成后处理，这种方式的优点在于代码容易写也容易读，后处理在描述页面状态和操作动作上都有优势，状态的顺序表现在代码顺序上而不是代码逻辑上，我认真的思考过真的没发现状态模式有什么优点， `if-else` 和 `switch` 是简单，难道我们为了现的有水平而抛弃简单的用复杂的吗？ 

另外 MVVM 类的框架js内存数据和Dom的同步以及数据改变时的 `watch` 监听才是状态思想的正确实现方式，但在实现上却是代理和发布-订阅模式的结合。

## 适配器模式

适配器模式和代理模式在实现上其实是一回事，都是包装模式，只是在函数功能上做了划分，适配只处理数据的格式，目的是兼容接口，代理是为了控制对对象的访问，还可以添加一些其他的处理逻辑。

## 附 - 设计原则

单一原则(SRP)

最少知识原则(LKP)

开放封闭原则(OCP)