Title:  Translate thread as 主题 is a bad idea  
Date:   Oct 18, 2022  
Tags:   raspberry_pi scanner

# Translate `thread` as `主题` is a bad idea.  

`主题` means a topic, a subject or a theme in simplified Chinese. And it has already been used as a translation for `theme` in [mattermost-webapp](https://translate.mattermost.com/browse/mattermost/mattermost-webapp_master/zh_Hans/?q=theme&sort_by=-priority%2Cposition&offset=3)/[mattermost-mobile](https://translate.mattermost.com/browse/mattermost/mattermost-mobile_master/zh_Hans/?offset=1&q=theme&sort_by=-priority%2Cposition&checksum=) translation project. It's just confusing.

### 那应该翻译成什么呢？

Mattermost 的`thread`和 Slack 的`thread`是一个东西，Slack 的中文简体译作`消息列`，繁体译作`對話串`。简体的翻译很别扭，繁体的翻译我觉得是可以接受的。  
而[mattermost-webapp/Chinese (Traditional)](https://translate.mattermost.com/projects/mattermost/mattermost-webapp_master/zh_Hant/)、
[mattermost-mobile/Chinese (Traditional)](https://translate.mattermost.com/projects/mattermost/mattermost-mobile_master/zh_Hant/)以及
[mattermost-server/Chinese (Traditional)](https://translate.mattermost.com/projects/mattermost/mattermost-server_master/zh_Hant/)有大部分的`thread`还没有翻译，已翻译的有`回覆串`、`訊息串`、`討論串`。
[mattermost-server/Chinese (Simplified)](https://translate.mattermost.com/browse/mattermost/mattermost-server_master/zh_Hans/?offset=1&q=%E8%AE%A8%E8%AE%BA%E4%B8%B2&sort_by=-priority%2Cposition&checksum=)中有一条`讨论串`，其余都是`主题`。  
总体来说比较混乱。当然我这里只讨论中文简体(Simplified Chinese)，引用繁体只是借鉴。

这个`thread`很难翻译么？其实不然，只要明白了`thread`在这里的真正含义，问题就会迎刃而解。

在英汉词典里查`thread`，看到最多的是**线**、**穿过**之类的字眼。的确，`thread`的本意就是**filament**，即线、穿针引线。也难怪人们要翻译成`消息列`、`對話串`之类，通过近义的`列`、`串`来和原文`thread`产生联系。  
但我在想，这种联系是必要的么？人们在翻译这个词的时候是否忽略了 Mattermost/Slack 为什么要把这个功能称为`thread`？

在"线"的基础上，`thread`有了很多引申义，其中最主要的一个就是*something continuous or drawn out*[^meaning]，连续的东西，一连串的东西。于是进入网络时代后，自然而然它就被用来代指BBS、论坛、电子邮件等载体中一系列的相关信息，即中文所谓的帖子。

Oxford 相关释义:
> a series of connected messages on email, social media, etc. that have been sent by different people

Merriam-Webster 相关释义:
> a series of electronic messages (as on a message board or social media website) following a single topic or in response to a single message

以下摘自 Wikipedia 词条 [Internet forum](https://en.wikipedia.org/wiki/Internet_forum)
> a single conversation is called a "thread", or topic.

> Within a forum's topic, each new discussion started is called a thread and can be replied to by as many people as so wish.

当然现在更多是叫`topic`，但许多老牌的论坛用的是`thread`，比如开源论坛程序[mybb](https://community.mybb.com/)，比如[LinuxQuestions](https://www.linuxquestions.org/questions/)。

这么说将其翻译成`主题`好像也没毛病。  
但像我开头所说，it's confusing。问题在于冲突和歧义。[mattermost-webapp](https://translate.mattermost.com/browse/mattermost/mattermost-webapp_master/zh_Hans/?offset=1&q=%E9%A3%8E%E6%A0%BC&sort_by=-priority%2Cposition&checksum=)上有部分翻译为了避免冲突，将`theme`翻译成了`风格`，但大部分仍然还是`主题`。我们都知道在显示样式上`主题`用的更广泛。所以建议保留`theme`的翻译为`主题`。

而`thread`，我认为有两个选项：
1. 可以借鉴 Slack 繁体中文的翻译——`对话串`，或者就是的`对话`。对话——两个或更多的人之间的谈话。也算是对`thread`的功能的另一种描述。当然，`讨论`也不错。
2. `跟帖`，本土化，和功能很贴切，中文用户都很熟悉[^gentie]，一看便能猜到意思。

I prefer the 2nd.

That's my two cents.

I'm posting it here because I think it needs to be discussed. Not just suggest in [Mattermost Weblate](https://translate.mattermost.com/) and maybe nothing would happen.

[^meaning]: [Definition of thread, Merriam-Webster](https://www.merriam-webster.com/dictionary/thread)
[^gentie]: [国家互联网信息办公室关于《互联网跟帖评论服务管理规定（修订草案征求意见稿）》公开征求意见的通知](http://www.cac.gov.cn/2022-06/17/c_1657089000974111.htm)
