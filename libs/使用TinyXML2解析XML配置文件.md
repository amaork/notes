# 使用TinyXML2解析XML配置文件


## 一、说明

TinyXML2是一个开源的以DOM方式解析XML文件的C++库，利用这个库的特性在这个库的基础上构造了两个C++类，然后使用这两个类来解析XML配置文件。

### 类设计

- XML\_PARSER:    XML解析器，根据添加的XML解析规则来解析XML文件
- XML\_PARSE\_RULE: XML解析规则，用来告知XML解析器，应该如何来解析XML文件

### 名词解释：

- XML节点：XML节点是XML中：<>与</>构成一个节点
- XML节点KEY:KEY是<>里的的内容被称为KEY,例如：节点2的KEY为“2”，值为“two”,节点3的KYE为“3”，值也是“3”；
- XML节点VALUE:在<></>之间的内容就是这个节点的VALUE，在一个XML节点中KEY与VALUE是成对出现的，参考下面的XML例子；
- XML解析器根节点：XML是一个多层嵌套的关系，在一个嵌套层里可以有多个同级XML节点，那么这些同级XML节点的上级节点就称为是这些节点的根节点例如：XML节点2,3，的根节点就是1.

        例子：
    
        <1>

        <2>two</2>

        <3>3</3>

        </1>
    

## 二、类设计

### 1、XML解析规则类

XML解析规则是由**XML\_PARSE\_RULE**类来实现的，其基本的设计思路是：告知XML解析器当前的这个XML节点中的KEY的内容是什么，以及这个KEY对应的VALUE保存在哪里？ 以及对应的VALUE是什么类型的（目前XML\_PARSER仅支持数字型的和字符串型的）

    XML_PARSE_RULE

    构造函数：
        XML_PARSE_RULE(){i_value = NULL, c_value = NULL, is_str = false;}    /*    默认 */
        XML_PARSE_RULE(const char *k, unsigned int *v) {key = k, i_value = v, is_str = false;}/*KEY 对应的VALUE是字符串*/
        XML_PARSE_RULE(const char *k, unsigned char *v){key = k, c_value = v, is_str = true;} /*KEY 对应的VALUE是数字*/

    属性：
        string key;                            /*    XML节点的KEY，根据这个在XML规则的的根节点中来查找XML节点*/
        bool is_str;                        /*    XML节点的VALUE是否是个字符串，如果是设置为TRUE */
        unsigned int *i_value;                /*    XML节点的整型VALUE的值存储到这里 */
        unsigned char *c_value;                /*    XML节点的字符型VALUE的值存储到这里 */


    方法：
        bool check(void);                    /*    检查该条RULE是否是有效的规则 */
        void print(ostream &os = cout）;        /*    打印该条规则在指定的流上，默认在标准输出上 */


### 2、XML解析器类

XML解析器是由**XML\_PARSER**类实现的，其基本思路是：一个解析器是由若干个解析规则组成的链表，这些解析规则共同使用一个XML根节点，也就是说这些规则只在这个根节点下有效，然后解析器在ROOT节点下，依次按照解析规则链表查找规则中的XML KEY然后将KEY对应的VALUE保存在规则制定的指针中。

并且解析器还提供了XML节点统计的功能，用来统计在解析过程中，一个或若干个根节点的出现次数，这个功能是针对具体的功能提供的，例如用来统计用户配置文件中红外码的个数等功能。

    XML\_PARSER
    构造函数：
        XML_PARSER(const string &desc, bool debug = false);    /*    构造一个XML解析器并设置解析器名称 */
        XML_PARSER(const string &desc, XMLElement *node, bool debug = false); /* 构造一个XML解析器并设置名称和解析根节点 */
    
    属性：
        XMLElement *root;                        /*    XML解析器的根节点，所有的解析规则共用一个根节点    */
        bool debug_option;                        /*    XML解析器的调试选项，如果打开将会输出解析规则和过程在指定的调试输出流上    */
        string parser_desc;                        /*    XML解析器的名称或描述    */
        vector< XML_PARSE_RULE> cnt_rules;        /*    XML解析器的统计规则链表，用来统一某个或若干个节点出现的次数    */
        vector< XML_PARSE_RULE> parser_rules;    /*    XML解析器规则链表，用来告知解析器应该如何解析XML节点 */

    私有方法：
        bool check(void);                        /*    检查XML解析器是否是有效的 */
        bool cnt_keys_check(const string &key);    /*    检查XML解析器当前解析的节点是否是要统一的节点，如果是则进行统计计数*/

    共有方法：
        bool parse(bool debug = false, ostream &debug_os = cerr);    /*    解析器主函数，调用来完成XML文件解析 */
        void print(int &flag, ostream &os = cout);                    /*    在指定的文件输出流上输出解析器的解析规则 */
        void set_debug(bool st) {debug_option = st; }                /*    设置解析器的调试开关 */
        void set_root(XMLElement *node){ root = node;}                /*    设置解析器解析的XML ROOT节点*/
        void set_desc(const string &desc) {parser_desc = desc;}        /*    设置解析器的名称和描述信息 */
        void add_rule(const XML_PARSE_RULE &rule);                    /*    添加一条XML解析规则到解析器解析链表 */
        void add_rule(const char *key, unsigned int *value);        /*    添加一条VALUE为整型的解析规则到解析器链表 */
        void add_rule(const char *key, unsigned char *value);        /*    添加一条VALUE为字符型的解析规则到解析器链表 */
        void add_cnt_rule(const char *key, unsigned int *value);    /*    添加一个统计规则到解析器统计链表 */


## 三、使用说明

要使用**XML\_PARSER**解析XML文件，首先要创建一个**XML\_PARSER**类，然后执行以下操作：

1. 首先，调用<font color=#0000ff>*add\_rule*</font>系列函数，向解析器中添加若干的解析的规则
2. 可选，调用<font color=#0000ff>*add\_cnt\_rule*</font>添加解析统计规则（可选，如果没有统计的需求可以不设置）
3. 然后，调用<font color=#0000ff>*set\_root*</font>函数设置解析器的ROOT节点，也就是开始解析的位置
4. 最后，调用<font color=#0000ff>*parse*</font>函数来完成XML的解析，解析后KEY对应的VALUE保存在规则中指针所指向的内存

使用伪代码描述这个过程就是：

    /*    创建一个XML解析器 */    
    xml_parser = new XML_PARSER("deom_parser");

    /*    向XML解析器中添加解析规则 */
    loop:

        xml_parser->add_rule(XML_KEY, &int_value);
        or 
        xml_parser->add_rule(XML_KEY, char_value_pointer);


    /*    设置解析统计（可选）*/
    xml_parser->add_cnt_rule(XML_CNT_KEY, &cnt_value);    

    /*    设置XML解析器解析跟节点 */
    xml_parser->set_root(xml_root_node);

    /*    开始解析配置XML文件    */
    ret = xml_parser->parse();

    /*    检查解析结果 */
    if (ret){
    
        error_process();
    }

    /* 删除解析器 */
    delete xml_parser;
    

**<font color = #ff0000>注意事项</font>**:

> - XML解析规则在调用解析器一次之后就自动删除了，因为每条规则的KEY对应一个VALUE，如果多次调用就会覆盖之前的解析结果
- XML解析器至少需要一条解析规则才能工作，并且在解析之前必须先设置解析器的解析的ROOT XML节点
- XML解析器解析的顺序的倒序的，即先添加的XML解析规则最后执行
- XML解析器可以将解析后的结果以及解析器中所有的解析规则输出到一个文件流中供调试使用，需要在print函数和parse函数中执行输出的流，同时打开调试功能

## 四、使用场景和例子

### 1.解析不重复的XML节点

XML文件中的配置不重复的情况下只需添加一个若干的解析规则解析一次即可，解析的结果按照规则的设置保存在指定的位置
    
    XML_PARSER xml_parser("Global", xml, debug);

    /* XML parser rules */
    xml_parser.add_rule(XML_GL_BOARD_NAME,         conf->global.board_name);
    xml_parser.add_rule(XML_GL_IR_USER_ID,        &conf->global.ir_user_id);    
    xml_parser.add_rule(XML_GL_IR_POWER_CODE,    &conf->global.ir_power_code);
    xml_parser.add_rule(XML_GL_POWER_ON_DELAY,    &conf->global.power_on_delay);

    /* Parse xml follwow parser rules  */
    if (!xml_parser.parse(debug, debug_os)){

        __DEBUG_MESG__("Parser global args failed!");
    }

### 2.解析重复的XML节点

XML配置文件中经常有重复的配置项，这些配置项使用相同的XML KEY标签，但对应的VALUE却是不同的，需要将这些VALUE分别保存到不同的位置如一个结构体数组中等。

另外还有一些配置项是和位置相关的，也就是说在解析的时候需要将这些值保存到数组的特定位置中去，而这些位置的信息也在XML配置文件中。这也就意味要达成这样的配置目标，需要首先在XML配置文件中解析对应的的位置信息，然后再解析其他数据将它们存放在特定位置。这样解析的过程就分成了两步：

- 第一步：解析出需要的全局位置信息
- 第二步：解析后续的数据，将这些数据保存在指定的位置

**例子：**

    /*    循环控制，解析不同的根节点下的重复的XML节点 */
    for (leaf = root->FirstChildElement(XML_KEYV_SUB_KEY); leaf; leaf = leaf->NextSiblingElement(XML_KEYV_SUB_KEY)){

        /*    创建一个新的XML解析器并设置解析器名称和根节点   */    
        keyv_parser = new XML_PARSER("KeyVoltage", leaf);

        /*    设置第一步需要的全局位置信息的解析规则  */
        keyv_parser->add_rule(XML_KEYV_ID,        &keyv_id);    

        /*    解析出全局位置信息供以后使用  */
        if (!keyv_parser->parse()){

            __DEBUG_MESG__("Parse Key voltage global data error!");
            goto out;
        }

        /*    添加通用的解析规则，将解析后对应的值保存到全局位置keyv_id指定的位置 */
        keyv_parser->add_rule(XML_KEYV_ID,        &conf->global.power.data.keyv.data[keyv_id].id);    
        keyv_parser->add_rule(XML_KEYV_MES,     &conf->global.power.data.keyv.data[keyv_id].mes_val);
        keyv_parser->add_rule(XML_KEYV_RATED,    &conf->global.power.data.keyv.data[keyv_id].value);
        keyv_parser->add_rule(XML_KEYV_LIMIT,    &conf->global.power.data.keyv.data[keyv_id].dev);
        keyv_parser->add_rule(XML_KEYV_DESC,    conf->global.power.data.keyv.data[keyv_id].desc);

        /*    打印解析器解析规则 */
        keyv_parser->print(print_flag, debug_os);

        /*    开始解析XML配置  */
        if (!keyv_parser->parse(debug, debug_os)){

            __debug_mesg__("Parser KeyVoltage id[%d] error!\n", keyv_id);
            goto out;
        }

        out:
        /*    删除解析器 */
        delete keyv_parser;
    }