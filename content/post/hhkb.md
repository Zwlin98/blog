---
title: "HHKB and Key bindings"
date: 2022-05-27T09:48:34+08:00
---

## HHKB

自 2021 年 4 月中购入 `HHKB`，到目前为止已经使用了一年多，期间除了偶尔用用笔记本自带的键盘外，没碰过别的键盘，总的来说，我对这把键盘很满意。对我来说，键盘的手感这种类玄学说法在我这里不生效，HHKB 对我来说就是一个完全够用的小个头键盘，比较合理的键位配置，舒服的移动距离，才是我喜欢它的原因。而我的快捷键配置也是随着我对 `HHKB` 的熟悉慢慢变化的，前半年我用的是有刻的键帽，后半年我开始尝试了无刻，然后就真香了。无刻虽然开始时比较困难，但是一旦熟悉，就只有好处了。

### 无刻的好处

+ 可以让注意力集中，不会因为低头看键盘找键而分心 (因为键盘上也没东西)
+ 可以迫使自己对键盘进行记忆，我相信大部分人对于字母可以盲打，但是对于部分符号，就比较困难。无刻可以帮助你记忆这些快捷键的位置，因为没有眼睛的帮助，你就没有退路，必须靠肌肉记忆。这也算刻意练习的一种吧。
+ 可以更好的自定义你的键盘，当你把一些键映射成别的键之后，键盘上印刷的功能和实际的功能并不相同，有时会突然会造成眼睛和肌肉记忆的冲突，而无刻就没有这种问题，一切键映射都存在于你的肌肉记忆中。
+ 配合你自己的各种键映射，可以想怎么舒服怎么来 (但是也加大了别人使用你电脑，和你使用别人电脑的难度

## 键映射

首先我会把系统的快捷键全部取消，因为那些功能我用不到，用到时可以再设置，这样你绑定自己的快捷键时就不用担心会不会冲突了。因此，系统的快捷键我只保留了一个切换输入法，其他的全部取消。

然后使用 [karabiner-elements](https://karabiner-elements.pqrs.org/) 进行简单的键映射：

| 键（组合键）  | 功能（映射）                                                 |
| ------------- | ------------------------------------------------------------ |
| Right Command | 单独按还是 Command，当和别的键「KEY」一起按时，映射成 Control + Option + Comand + 「KEY」 |
| Right Option  | 映射成 Comma，也就是逗号键，用作我 vim 的第二个 Leader（第一个 Leader 是空格） |
| Left Control  | 单独按是切换输入法，当和别的键「KEY」一起按时，映射成 Control  + 「KEY」 |
| Tab + hjkl    | 映射成方向键                                                 |

`Right Command` 和 `Right Option` 在我个人的体验中，原本的功能我几乎用不到，都用的是左边的，与其让其闲置，不如将其映射成别的键，其中 Right Command 映射成 `Control + Option + Comand`，可以作为我个人的一个 super prefix 键，以保证不会和任何软件或者系统的快捷键冲突。

上述功能的达成，需要用到 `Karabiner` 的 `Complex Modifications` 功能，网上教程很多，也可以使用别人分享出来的方案，我这里就不多讲述了。

这里多提一句，因为 HHKB 的 `Control` 是放在传统键盘 `Caps Lock` 的位置的，所以 mac 自带的 `Caps Lock` 如果被映射成了 `Control`，也是可以用这个方案来保留 **Caps Lock** 切换输入法的功能的。下面放上我完成这个功能的 `Karabiner` 配置：

```json
{
  "title": "swith input with ctrl",
  "rules": [
    {
      "description": "短按 ctrl 切换输入法",
      "manipulators": [
        {
          "from": {
            "key_code": "left_control"
          },
          "to": [
            {
              "key_code": "left_control"
            }
          ],
          "to_if_alone": [
            {
              "key_code": "spacebar",
              "modifiers": [
                "left_option",
                "left_command"
              ]
            }
          ],
          "type": "basic"
        }
      ]
    }
  ]
}
```

**PS**：我这里并不是把短按 `Control` 变成 `Caps Lock`，而是变成了 `Command + Option + Space`。这是因为我不喜欢 `Caps Lock` 的功能，所以为了避免按成 `Caps Lock`，单独映射到了一个组合键 (这个组合键是我在系统里配置的切换输入法的组合键，也可以变成别的，但是还是不建议变成 Caps Lock，容易按错)。

## 超级键

有了这个超级键 `Control + Option + Comand` 之后，你就可以用这个 Super Prefix 来配置自己的快捷键，例如使用 Thor，Ray Cast，HammerSpoon 等软件，映射自己的快捷键。

| 组合键（Super Prefix 用 SP 指代） | 功能                              |
| ---------------------------------- | --------------------------------- |
| SP + ` (反单引号，HHKB 最右上角）   | 切换到终端（iterm2）              |
| SP + d                             | 使用 dash 搜索（用 raycast 完成）    |
| SP + v                             | 从剪贴板历史粘贴（用 raycast 完成） |
| SP + 1                             | 打开 Chrome                       |
| …                                  | …                                 |



