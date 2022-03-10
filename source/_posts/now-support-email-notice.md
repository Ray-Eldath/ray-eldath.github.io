---
title: 本博客评论区现已支持邮件提醒
date: 2022-03-10
toc: false
categories: notice
excerpt: " "
---

[Valine](https://valine.js.org/) 作为一个无后端、轻量简洁开放的评论插件确实好用，然而也正是因为这简单轻量的评论系统，本博客的评论一直没有通知功能。很长一段时间内我都靠「人肉轮询」来处理回复，经常很久以后才看到可爱读者们留言，而我的回复也难以传达。今天终于决心解决这个问题，原本想着跑个 [Valine-Admin](https://github.com/DesertsP/Valine-Admin) ，然而恰逢我使用的 [hexo 主题](https://github.com/ppoffice/hexo-theme-icarus)（魔改了一些 CSS）终于支持了 [Waline](https://waline.js.org/) 这个进化自 Valine、拥有更好的界面和更多功能且**原生支持邮件提醒**的评论插件，在用 sed（甚至写进了本博客使用的 GitHub Actions pipeline 里边...）更新了这 hexo 主题依赖的老旧无比的 Waline 版本之后，这儿终于有带提醒的评论区了。

唯一一点遗憾是，即便我已按照 GandiMail（本域名的服务商 Gandi.net 附赠的域名邮箱服务）的要求设置了 SPF 记录等各种 DNS 条目，本博客的评论提醒邮箱 `no-reply@ray-eldath.me` 还是会被 Outlook 归为「垃圾邮件」。我们 Ray 真是太可怜了，希望这只是个个例。也请此后在这留言的各位读者们更加勤快地查看自己的垃圾箱 ~~（看看有没有 Ray）~~。

希望我的拖延症能快点好。