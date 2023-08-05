---
layout: '../../layouts/MarkdownPost.astro'
title: '[C++项目] Boost文档 站内搜索引擎(4): 实现搜索的相关接口、线程安全的单例index接口、cppjieba分词库的使用...'
pubDate: 2023-08-05
description: '本篇文章的内容为: 查找、搜索 相关接口的实现, 建立索引接口的相关优化, 本地搜索测试. 做完上面的内容, 就后面就是加入网络和页面的制作了~'
author: '七月.cc'
cover:
    url: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308050919612.png'
    square: 'https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308050919612.png'
    alt: 'cover'
tags: ["C++", "项目", "Linux", "Boost"]
theme: 'light'
featured: false
---

![|cover](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308050919612.png)

---

有关`Boost`文档搜索引擎的项目的前三篇文章, 已经分别介绍分析了:

1. 项目背景: [🫦[C++项目] Boost文档 站内搜索引擎(1): 项目背景介绍、相关技术栈、相关概念介绍...](https://www.julysblog.cn/posts/Boost-Doc-Searcher-I)
2. 文档解析、处理模块`parser`的实现: [🫦[C++项目] Boost文档 站内搜索引擎(2): 文档文本解析模块parser的实现、如何对文档文件去标签、如何获取文档标题...](https://www.julysblog.cn/posts/Boost-Doc-Searcher-II)
3. 文档 **正排索引与倒排索引** 建立的接口的实现: [🫦[C++项目] Boost文档 站内搜索引擎(3): 建立文档及其关键字的正排 倒排索引、jieba库的安装与使用...](https://www.julysblog.cn/posts/Boost-Doc-Searcher-III)
4. **`建议先阅读上面三篇文章`**

已经实现了对文档建立索引的相关接口. 有了接口, 就可以调用并建立文档索引.

建立了索引, 其实就可以根据索引查找文档了. 所以, 本篇文章的内容即为: 

1. 查找、搜索 相关接口的实现
2. 建立索引接口的相关优化
3. 本地搜索测试

做完上面的内容, 就后面就是加入网络和页面的制作了~

# 搜索

搜索是通过输入的内容进行搜索的. 并且一定是 **先在倒排索引中找到文档`id`, 再根据文档`id`去正排索引中找到文档** 的内容.

而倒排索引中存储的内容是对文档内容进行分词, 然后根据分词建立的.

那么要实现搜索, 也需要 **对搜索的内容进行分词, 然后再根据搜索内容的分词 在 倒排索引中查找关键词对应的倒排拉链**

## 搜索接口的基本结构

了解了搜索的流程, 那么搜索的相关接口的基本结构实际也就显现出来了:

```cpp
namespace ns_searcher {
	class searcher {
	private:
		ns_index::index* _index; // 建立索引的类

	public:
        // 初始化接口
        // 在搜索之前需要先建立索引. 这个接口就是建立索引用的
		void initSearcher(const std::string& input) {}

		// 搜索接口
		// 搜索需要实现什么功能?
        // 搜索需要接收字符串, 然后针对字符串进行分词 再根据分词在索引中进行查找
		// 首先参数部分需要怎么实现?
		// 参数部分, 需要接收需要搜索的句子或关键字, 还需要一个输出型参数 用于输出查找结果
		//  查找结果我们使用jsoncpp进行序列化和反序列化
		void search(const std::string& query, std::string* jsonString) {}
```

基本的结构就这么简单. 只需要对外提供两个接口:

1. `initSearcher()` 初始化接口
2. `search()` 搜索接口

## `initSearcher()`接口 实现

`initSearcher()` 是用来做搜索前的工作的, 实际就是建立索引的接口

但是, 在建立索引之前 我们清楚 所有的搜索都是在唯一一个倒排索引和唯一一个正排索引中进行的. 也就是说 **最终一个程序中只需要建立一次索引**. 所以我们可以将索引的相关函数实现为单例.

### `index`接口类 单例实现

`index`类的单例实现非常的简单:

```cpp
namespace ns_index {

	// 用于正排索引中 存储文档内容
	typedef struct docInfo {
		std::string _title;	  // 文档标题
		std::string _content; // 文档去标签之后的内容
		std::string _url;	  // 文档对应官网url
		std::size_t _docId;	  // 文档id
	} docInfo_t;

	// 用于倒排索引中 记录关键字对应的文档id和权重
	typedef struct invertedElem {
		std::size_t _docId;	   // 文档id
		std::string _keyword;  // 关键字
		std::uint64_t _weight; // 搜索此关键字, 此文档id 所占权重

		invertedElem() // 权重初始化为0
			: _weight(0) {}
	} invertedElem_t;

	// 关键字的词频
	typedef struct keywordCnt {
		std::size_t _titleCnt;	 // 关键字在标题中出现的次数
		std::size_t _contentCnt; // 关键字在内容中出现的次数

		keywordCnt()
			: _titleCnt(0)
			, _contentCnt(0) {}
	} keywordCnt_t;

	// 倒排拉链
	typedef std::vector<invertedElem_t> invertedList_t;

	class index {
	private:
		// 正排索引使用vector, 下标天然是 文档id
		std::vector<docInfo_t> forwardIndex;
		// 倒排索引 使用 哈希表, 因为倒排索引 一定是 一个keyword 对应一组 invertedElem拉链
		std::unordered_map<std::string, invertedList_t> invertedIndex;

		// 单例模式设计
		index() {}

		index(const index&) = delete;
		index& operator=(const index&) = delete;

		static index* _instance; // 单例
		static std::mutex _mtx;

	public:
		// 获取单例
		static index* getInstance() {
			if (nullptr == _instance) {
				_mtx.lock();
				if (nullptr == _instance) {
					_instance = new index;
				}
				_mtx.unlock();
			}

			return _instance;
		}
		
        // 通过关键字 检索倒排索引, 获取对应的 倒排拉链
		invertedList_t* getInvertedList(const std::string& keyword) {}

		// 通过倒排拉链中 每个倒排元素中存储的 文档id, 检索正排索引, 获取对应文档内容
		docInfo_t* getForwardIndex(std::size_t docId) {}

		// 根据parser模块处理过的 所有文档的信息
		// 提取文档信息, 建立 正排索引和倒排索引
		// input 为 ./data/output/raw
		bool buildIndex(const std::string& input) {}

	private:
		// 对一个文档建立正排索引
		docInfo_t* buildForwardIndex(const std::string& file) {}
        // 对一个文档建立倒排索引
		bool buildInvertedIndex(const docInfo_t& doc) {}
	};
	// 单例相关
	index* index::_instance = nullptr;
	std::mutex index::_mtx;
}
```

需要做的工作也就只有:

1. 添加两个成员变量, 并在类外定义: 

    **`static index* _instance;`**

    **`static std::mutex _mtx;`**

2. 构造函数设置私有, 拷贝构造函数和赋值重载函数删除:

    **`index() {}`**

    **`index(const index&) = delete;`**

    **`index& operator=(const index&) = delete;`**

3. 添加线程安全的获取单例的公开接口:

    ```cpp
    static index* getInstance() {
        if (nullptr == _instance) {
            _mtx.lock();
            if (nullptr == _instance) {
                _instance = new index;
            }
            _mtx.unlock();
        }
    
        return _instance;
    }
    ```

这样就将`index`类设计为了单例模式

### 接口实现

`initSearcher()`接口的实现也是非常的简单, 只需要建立索引就可以了:

```cpp
void initSearcher(const std::string& input) {
    // 搜索前的初始化操作
    // search类成员 ns_index::index* _index 获取单例
    _index = ns_index::index::getInstance();
    std::cout << "获取单例成功 ..." << std::endl;
    
    // 建立索引
    _index->buildIndex(input);
    std::cout << "构建正排索引、倒排索引成功 ..." << std::endl;
}
```

## `search()`接口 实现

`searcher`类中, 初始化接口`initSearcher()`实现的简单.

但是`search()`就没有那么简单了, 需要注意非常多的细节

搜索接口需要实现的功能是: 

1. 接收字符串, 然后针对字符串进行分词
2. 再根据分词在倒排索引中查找对应的倒排拉链
3. 通过倒排拉链获取相关文档的id
4. 再根据文档id, 查找正排索引查找对应的文档内容信息
5. 最终查找到的文档内容信息是需要输出的, 所以我们接口使用了输出型参数

但这只是功能实现的整体逻辑. 还有许多的细节需要考虑:

1. 倒排索引中的 关键词都是小写的, 而搜索输入的内容很可能存在大小写, 如何实现忽略大小写的搜索呢?

2. 查找到倒排拉链之后, 是可以通过遍历拉链 获取到文档id等相关信息的

    不过, 页面的显示是需要按照相关度排序的, 我们也在倒排索引中 使用词频简单地体现出了 关键字与对应文档的相关性

    那么如何对获取到的文档进行排序呢?

3. 在查找的时候, 一定会有不同的词 查找到同一个文档的问题. 那么 如果不做处理, 就会出现同一个文档在页面中不同的位置 被显示出来的问题, 该怎么解决呢?

4. 获取到文档内容信息之后, 是需要将设置文档需要展示的相关信息的: `title` `description` `url`

    如果文档内容过长, 一定不能将文档全部内容展示在搜索页面中, 那么如何获取文章相关的摘要呢?

5. 还有一些其他细节, 结合代码具体分析...

那么, 根据需求 `search()`接口的实现代码就是这样的:

```cpp
typedef struct invertedElemOut {
    std::size_t _docId;
    std::uint64_t _weight;
    std::vector<std::string> _keywords;
} invertedElemOut_t;

// 搜索接口
// 首先参数部分需要怎么实现?
// 参数部分, 需要接收需要搜索的句子或关键字, 还需要一个输出型参数 用于输出查找结果
//  查找结果我们使用jsoncpp进行序列化和反序列化
// search() 具体需要实现的功能:
//  1. 对接收的句子或关键词进行分词
//  2. 根据分词, 在倒排索引中查找到所有分词的倒排拉链 并汇总其中的 invertedElem, 然后根据相关性进行排序
//  4. 然后再遍历所有的 invertedElem, 根据 invertedElem中存储的 文档id, 在正排索引中获取到文档内容
//  5. 然后将获取到的文档内容使用jsoncpp 进行序列化, 存储到输出型参数中
// 直到遍历完invertedElem
void search(const std::string& query, std::string* jsonString) {
    // 1. 对需要搜索的句子或关键词进行分词
    std::vector<std::string> keywords;
    ns_util::jiebaUtil::cutString(query, &keywords);

    std::vector<invertedElemOut_t> allInvertedElemOut;

    // 统计文档用, 因为可能存在不同的分词 在倒排索引中指向同一个文档的情况
    // 如果不去重, 会重复展示
    std::unordered_map<std::size_t, invertedElemOut_t> invertedElemOutMap;
    // 2. 根据分词获取倒排索引中的倒排拉链, 并汇总去重 invertedElem
    for (std::string word : keywords) {
        boost::to_lower(word);

        ns_index::invertedList_t* tmpInvertedList = _index->getInvertedList(word);
        if (nullptr == tmpInvertedList) {
            // 没有这个关键词
            continue;
        }

        for (auto& elem : *tmpInvertedList) {
            // 遍历倒排拉链, 根据文档id 对invertedElem 去重
            auto& item = invertedElemOutMap[elem._docId]; // 在map中获取 或 创建对应文档id的 invertedElem
            item._docId = elem._docId;
            item._weight += elem._weight;
            // 权重需要+= 是因为多个关键词指向了同一个文档 那么就说明此文档的与搜索内容的相关性更高
      		// 就可以将多个关键字关于此文档的权重相加, 表示搜索相关性高
            // 最好也将 此文档相关的关键词 也存储起来, 因为在客户端搜索结果中, 可能需要对网页中有的关键字进行高亮
            // 但是 invertedElem 的第三个成员是 单独的一个string对象, 不太合适
            // 所以, 可以定义一个与invertedElem 相似的, 但是第三个成员是一个 vector 的类, 比如 invertedElemOut
            item._keywords.push_back(elem._keyword);
            // 此时就将当前invertedElem 去重到了 invertedElemMap 中
        }
    }
    // 出循环之后, 就将搜索到的 文档的 id、权重和相关关键词 存储到了 invertedElemMap
    // 然后将文档的相关信息 invertedElemOut 都存储到 vector 中
    for (const auto& elemOut : invertedElemOutMap) {
        // map中的second: elemOut, 在执行此操作之后, 就没用了
        // 所以使用移动语义, 防止发生拷贝
        allInvertedElemOut.push_back(std::move(elemOut.second));
    }

    // 执行到这里, 可以搜索到的文档id 权重 和 相关关键词的信息, 已经都在allInvertedElemOut 中了.
    // 但是, 还不能直接 根据文档id 在正排索引中检索
    // 因为, 此时如果直接进行文档内容的索引, 在找到文档内容之后, 就要直接进行序列化并输出了. 而客户端显示的时候, 反序列化出来的文档顺序, 就是显示的文档顺序
    // 但是现在找到的文档还是乱序的. 还需要将allInvertedElemOut中的相关文档, 通过_weight 进行倒序排列
    // 这样, 序列化就是按照倒序排列的, 反序列化也会如此, 显示同样如此
    std::sort(allInvertedElemOut.begin(), allInvertedElemOut.end(),
              [](const invertedElemOut_t& elem1, const invertedElemOut_t& elem2) {
                  return elem1._weight > elem2._weight;
              });

    // 排序之后, allInvertedElemOut 中文档的排序就是倒序了
    // 然后 通过遍历此数组, 获取文档id, 根据id获取文档在正排索引中的内容
    // 然后再将 所有内容序列化
    Json::Value root;
    for (auto& elemOut : allInvertedElemOut) {
        // 通过Json::Value 对象, 存储文档内容
        Json::Value elem;
        // 通过elemOut._docId 获取正排索引中 文档的内容信息
        ns_index::docInfo_t* doc = _index->getForwardIndex(elemOut._docId);
        // elem赋值
        elem["url"] = doc->_url;
        elem["title"] = doc->_title;
        // 关于文档的内容, 搜索结果中是不展示文档的全部内容的, 应该只显示包含关键词的摘要, 点进文档才显示相关内容
        // 而docInfo中存储的是文档去除标签之后的所有内容, 所以不能直接将 doc._content 存储到elem对应key:value中
        elem["desc"] = getDesc(doc->_content, elemOut._keywords[0]); // 只根据第一个关键词来获取摘要
        // for Debug
        // 这里有一个bug, jsoncpp 0.10.5.2 是不支持long或long long 相关类型的, 所以需要转换成 double
        // 这里转换成 double不会有什么影响, 因为这两个参数只是本地调试显示用的.
        elem["docId"] = (double)doc->_docId;
        elem["weight"] = (double)elemOut._weight;

        root.append(elem);
    }

    // 序列化完成之后将相关内容写入字符串
    // for Debug 用 styledWriter
    Json::StyledWriter writer;
    *jsonString = writer.write(root);
}
```

执行搜索, 首先要做的就是 **对传入的字符串进行分词**

然后根据每个分词, 在倒排索引中查找对应的倒排拉链, 再通过遍历倒排拉链就可以获取到当前关键字对应出现的文档相关信息.

不过, 分词之后-遍历时-正式查找之前 要做的首要任务就是, **将分词转换为小写**. 因为, 倒排索引中的所有关键词 都是小写的状态

并且, 查找到倒排拉链 在获取并统计文档信息时, 还会出现不同关键字指向同一文档的情况, 这种情况是需要处理的 **不能多次记录同一个文档**.

还有就是, 如果一次搜索中 **多个关键词指向了同一个文档 那么就说明此文档的与搜索内容的相关性更高**, 此时是需要将文档的显示权重增加的.

根据这些需求, 实现了第一部分的代码:

 ![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308052319950.png)

第一部分的代码实现了:

1. 对搜索内容分词
2. 遍历分词查找倒排拉链
3. 根据倒排拉链 去重获取文档信息

这部分代码, 有三个要点:

1. 需要定义一个`unordered_map`来实现对搜索到的文档 记录并去重

2. 如果单纯地 对多个关键词搜到的同一个文档 去重, 而不记录相关的关键字, 那么就无法得知此文档是根据那些关键字搜索到的.  那么再去重的同时, 还需要记录对应的关键词

    也就是说, `unordered_map` 存储的元素类型不能是简单的`ns_index::invertedElem`, 因为`invertedElem`没有办法很好的记录多个关键词

    所以, 定义了一个结构体:

    ```cpp
    typedef struct invertedElemOut {
        std::size_t _docId;
        std::uint64_t _weight;
        std::vector<std::string> _keywords;
    } invertedElemOut_t;
    ```

    成员依旧包括 文档`id`和权重, 但是第三个成员变量与`invertedElem`不同, `invertedElemOut`的第三个成员变量是`vector<string>`, 适合存储多个关键字.

3. 第三个要点就是: `unordered_map`中存储的对应此关键字的元素的权重, 需要`+=`当前关键字的权重.

    因为 **多个关键词指向了同一个文档 那么就说明此文档的与搜索内容的相关性更高, 所以 就可以将多个关键字关于此文档的权重相加, 表示搜索相关性高**

第一部分执行完之后, 根据搜索内容 查找到的所有的文档的相关信息, 都存储在了`invertedElemOutMap`中.

接下来要做的, 并不是遍历`unordered_map`获取文档`id`, 去正排索引中查找文档的内容. 而是需要先根据文档的显示权重进行排序. 排完序之后, 再进行文档内容的获取.

因为, 获取每到一个文档内容就需要将文档内容输出了, 输出之后 就要做处理响应回客户端进行显示了. 这也意味着 在正排索引中的查找顺序 实际就是搜索结果的显示顺序, 所以在查找之前, 需要先排序:

![](https://dxyt-july-image.oss-cn-beijing.aliyuncs.com/202308052342538.png)