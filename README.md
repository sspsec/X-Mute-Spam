# X-Mute-Spam · 黄推一键静音

> 一键自动把关键词添加到 X（Twitter）的 **「隐藏的字词」** 功能里。  
> 专门过滤评论区和时间线里的黄推、引流、约炮、网盘广告，尽量不影响正常关注的色情博主。

## 功能特点

- 自动添加 **144 个** 精选高危关键词到 X 的「隐藏的字词」
- 重点过滤：约炮、同城、加V、主页自取、网盘链接、常见引流话术
- 包含高频引流 emoji（🍑🍆💦🔥🔞👇👉 等）
- 支持断点续传（刷新页面可继续）
- 内置防 429 风控休息机制
- 纯前端脚本，无需安装任何扩展

## 使用方法

1. 打开 X 网页版，进入：  
   **设置 → 隐私和安全 → 隐藏和屏蔽 → 隐藏的字词**
2. 按 `F12` 打开开发者工具 → 切换到 **Console（控制台）**
3. 复制下方完整脚本，粘贴进控制台，回车运行
4. 脚本会自动把关键词一个个添加到「隐藏的字词」列表中，期间请勿关闭页面

<img width="1172" height="942" alt="file-3d2d0a7df3a0f9ae1cfd4518b6085ae8" src="https://github.com/user-attachments/assets/5d7d7dd1-d9b6-4a91-b445-3bfd8f338885" />

## 风控说明（重要）

脚本内置了防 429 机制，运行时会自动处理：

- **常规休息**：每成功添加 30 个关键词，会自动暂停 **3 分钟**，防止触发频率限制。
- **触发 429 风控**：如果 X 返回保存失败（429），脚本会自动退回，并暂停 **10 分钟**，然后从当前进度继续。
- 暂停期间请**不要关闭页面**，也不要手动操作，脚本会自己恢复。
- 进度会保存在浏览器本地（localStorage），刷新页面后也能从上次中断的地方继续。

整个过程预计需要一些时间（取决于是否触发风控），建议挂机运行，不用一直盯着。

## 完整脚本

请复制以下全部代码到控制台运行：

``````
(async function autoAddUltimateUnifiedVersion() {
  const keywords = [
    "加v","加vx","+v","+V","加微","十V","伽V","企鹅号","纸飞机","tg群","电报群","小飞机",
    "主页自取","主页领取","看主页","主页福利","主页置顶","必须主页看","快去我主页","点我头像",
    "主页有福利","主页有惊喜","用我主页","主页工具","传送门","看置顶","看简介",
    "防失联","防走丢","备用号","小号","内部群","内部吃瓜","吃瓜入口","资源群","吃瓜群",
    "夸克网盘","pan.quark","阿里云盘","迅雷网盘","蓝奏","网盘","云盘",
    "pan.quark.cn","drive.uc.cn","pan.xunlei.com","t.cn","t.cn/",
    "完整版视频","完整视频","无删减","高清无删减","网盘提取","解压密码","吃瓜链接",
    "资源","片子","片源","番号","免费看","吃瓜视频",
    "同城","线下","约炮","搭子","固炮","炮友","长期搭子","找搭子","上门约","同城空降","同城可约",
    "有哥哥线下吗","万达广场","酒店","宾馆","民宿","过夜","留宿","开房","面基",
    "出来吗","有空吗","能过夜","能出来","能见","能约","能线下",
    "附近","离得近","本地","当面","约吗","陪我",
    "不是人机","福不黑","不收费","互免","费破","破费","不收钱","不谈钱",
    "比我骚","比我涩","太涩","没她好看","好看的没我骚","没我好看",
    "比她好看的没她骚比她骚的没她好看","就她的主页能","就她主页能打了",
    "这个玩具太涩了","有人想和我一下吗","刷了半天","应该没人比我","长期搭子呀",
    "玩归玩闹归闹","给你看福我不开玩笑","不跟你开玩笑","给你看点刺激的",
    "不入生活","极品的主页","我福不黑","蹲一个男搭子","有人想和我","找个哥哥","找我玩",
    "成为我的人","小哥哥的","给我看福","看福我","的福","大福","小马","大车","闺蜜和",""
    "🍑","🍆","💦","🔥","💋","👅","🔞","😈","🥵","💯","👇","👉","🔗","📱"
  ];

  const uniqueKeywords = Array.from(new Set(keywords));
  const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

  window.__x_isRateLimited = false;
  const origFetch = window.fetch;
  window.fetch = async function(...args) {
    const url = typeof args[0] === 'string' ? args[0] : (args[0] && args[0].url ? args[0].url : '');
    if (url.includes('discouraged.json')) {
      return new Response('[]', { status: 200, headers: { 'Content-Type': 'application/json' } });
    }
    const response = await origFetch.apply(this, args);
    if (response.status === 429 && url.includes('create.json')) {
      window.__x_isRateLimited = true;
    }
    return response;
  };

  const origXHR = window.XMLHttpRequest.prototype.open;
  window.XMLHttpRequest.prototype.open = function(method, url, ...rest) {
    this.addEventListener('load', function() {
      if (this.status === 429 && url.includes('create.json')) {
        window.__x_isRateLimited = true;
      }
    });
    return origXHR.call(this, method, url, ...rest);
  };

  let currentIndex = parseInt(localStorage.getItem('x_muted_progress_unified') || '0', 10);

  if (currentIndex >= uniqueKeywords.length) {
    console.log('🎉 所有关键字已添加完毕！');
    return;
  }

  console.log(`🚀 启动无人值守挂机模式！共计 ${uniqueKeywords.length} 个词。`);
  console.log(`📊 当前进度: ${currentIndex} / ${uniqueKeywords.length}`);

  let countInCurrentBatch = 0;

  while (currentIndex < uniqueKeywords.length) {
    if (countInCurrentBatch > 0 && countInCurrentBatch % 30 === 0) {
      console.log(`✅ 本批次 30 个已达标，开始常规防风控休息 3 分钟...`);
      for (let m = 3; m > 0; m--) {
        console.log(`⏳ [常规休息] 还剩 ${m} 分钟...`);
        await wait(60 * 1000);
      }
      countInCurrentBatch = 0;
    }

    const word = uniqueKeywords[currentIndex];
    let addBtn = document.querySelector('a[href="/settings/add_muted_keyword"]');
    if (addBtn) {
      addBtn.click();
      await wait(1500);
    }

    let input = document.querySelector('input[name="keyword"]') || document.querySelector('input[type="text"]');
    if (!input) {
      await wait(2000);
      continue;
    }

    input.focus();
    input.click();
    document.execCommand('selectAll', false, null);
    document.execCommand('insertText', false, word);

    const nativeSetter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value")?.set;
    if (nativeSetter) nativeSetter.call(input, word);
    input.dispatchEvent(new Event('input', { bubbles: true }));
    input.dispatchEvent(new Event('change', { bubbles: true }));

    await wait(1000);

    let saveBtn = document.querySelector('[data-testid="settingsDetailSave"]');
    if (!saveBtn) {
      const allButtons = Array.from(document.querySelectorAll('div[role="button"]'));
      saveBtn = allButtons.find(b => b.innerText && b.innerText.includes('保存'));
    }

    window.__x_isRateLimited = false;
    if (saveBtn) {
      saveBtn.dispatchEvent(new MouseEvent('mousedown', { bubbles: true }));
      saveBtn.dispatchEvent(new MouseEvent('mouseup', { bubbles: true }));
      saveBtn.click();
    }

    await wait(2000);

    if (window.__x_isRateLimited) {
      console.warn(`🚨 触发了 X 保存接口的 429 限制！退回并等待 10 分钟...`);
      let backBtn = document.querySelector('[aria-label="返回"]') || document.querySelector('[data-testid="app-bar-back"]');
      if (backBtn) backBtn.click();

      for (let m = 10; m > 0; m--) {
        console.log(`⏳ [风控冷却] 还剩 ${m} 分钟... (期间请勿操作网页)`);
        await wait(60 * 1000);
      }
      window.__x_isRateLimited = false;
      continue;
    }

    console.log(`[${currentIndex + 1}/${uniqueKeywords.length}] ✅ 已保存: ${word}`);
    currentIndex++;
    countInCurrentBatch++;
    localStorage.setItem('x_muted_progress_unified', currentIndex);

    await wait(1500);
  }

  console.log('🎉 挂机结束！所有补充词汇已经安全、无遗漏地添加完毕。');
})();
``````



## 注意事项

- 脚本仅在你自己的浏览器本地运行，不会上传任何数据
- 建议添加时选择「主页时间线 + 通知」同时生效
- 如遇到风控（429），脚本会自动休息，请耐心等待
- 本项目仅用于过滤垃圾引流，请合理使用

## 免责声明

本脚本仅供学习交流使用，使用者需自行承担使用风险。  
请遵守 X 平台相关规定。

---

如果觉得有用，欢迎 Star ⭐  
有更好的关键词可以提 Issue。
