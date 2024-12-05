# Cocos 回合制游戏文档

游戏目前包含装备系统，资源系统，战斗系统，技能系统，存储系统，成就系统，BUFF系统，剧情系统

下文 根目录 指的是 assets 文件夹

## 目录结构

![图片](./README/01.png)

- Module **(第三方库文件)**
  - Axios **(前端http网络模块)**
  - CcNative **(Cocos本地功能封装)**
    - Asset **(资源管理)**
    - Storage **(本地存储)**
  - EventManager **(事件管理模块)**
  - Extension **(扩展模块)**
    - Component **(组件扩展)**
  - Rx **(响应式模块，源自Vue3源码，可以用来做发布订阅)**
    - reactivity **(响应式功能实现)**
    - shared **(共享模块)**

- Prefab **(预制体文件夹，可通过AssetManager类动态获取)**
  - MessageAlert **(消息展示预制体)**
  - UserResource **(用户资源展示预制体)**
- Resource **(资源文件夹，可通过AssetManager类动态获取)**
  - Animation **(动画Clip文件夹)**
  - Font **(字体文件夹)**
  - Images **(各种装备，物品，buff，角色的图标文件夹)**
  - Music **(音乐文件夹)**
  - Spine **(动画文件夹)**
  - Ui **(图标文件夹)**
- Scenes **(场景文件夹)**
- Script **(脚本文件夹，主要代码在这里)**
  - Component **(扩展组件)**
  - Data **(用户数据，游戏数据存储)**
  - Mod **(主要扩展代码，可以添加物品，装备，怪物，关卡...)**
  - Prefab **(对应 根目录/Prefab 文件夹下的所有预制体的脚本)**
  - Scenes **(场景组件)**
  - System **(游戏系统的实现)**
    - Base **(基本类)**
    - Caculate **(计算公式)**
    - Instance **(实例化的原型)**
    - Prototype **(基础原型)**
  - Util **(工具代码)**

## Module (第三方库模块)

提供一些常用的通用功能封装，用来简化功能开发，或者引入第三方库，亦或者进行某些通用扩展操作

### **Axios** (HTTP网络请求)

移植自Axios，具体用法可参考文档  [Axios中文文档](https://www.axios-http.cn/docs/intro) 目前web端完全可用，android，ios未测试

###  **Asset** (资源系统)

封装于cocos自带的AssetManager的资源管理API

#### CcAssetManager

该代码源文件位于 **根目录/Module/CcNative/Asset/AssetManager.ts**

AssetManager 构造函数的定义为

```typescript
class CcAssetManager {
    constructor(bundleName: string) {}
}
```

bundleName 为一个字符串参数，该字符串需要在编辑器中设置

​		![](./README/03.png)

选择一个文件夹，勾选 isBundle，输入BundleName，之后可以通过BundleName获取该文件夹下的资源

可以直接通过导入 CcNative 使用

示例: 

```typescript
import { CcNative } from "根目录路径/Module/CcNative";
// 实例化 AssetManager 类
const assetManager = new CcNative.Asset.AssetManager("Resource")

// 初始化后无需等待即可以直接进行资源加载

// load用法与cocos原有AssetManager.load用法类似，返回一个Promise
assetManager.load("资源路径" , SpriteFrame/** 资源类型 **/) 
.then(res => {
    // 如果不出意外 res 就是需要的资源
    log(res)
}).catch(e => {
    log("资源不存在")
})

// loadDir用法也与AssetManager.loadDir类似，返回一个Promise
assetManager.loadDir("文件夹路径") 
.then(res => {
    // 如果不出意外 res 就是需要的资源数组
    log(res)
}).catch(e => {
    log("资源不存在")
})
```

使用 CcAssetManager 会对资源进行缓存操作，加速下一次获取，相邻两次获取同一个资源也会进行节流合并: 

```typescript
const assetPath = "临时路径"
const p1 = assetManager.load(assetPath)
const p2 = assetManager.load(assetPath)
log(p1 === p2) // true 实际上只会获取一次
```

#### RemoteAsset

该代码源文件位于 **根目录/Module/CcNative/Asset/RemoteAsset.ts**

它只提供一个函数 **loadRemote** ，用于加载远程资源，用法与 cocos **AssetManager.loadRemote** 用法相同

```typescript
import { CcNative } from "根目录路径/Module/CcNative";

CcNative.Asset.loadRemote("http://xxx" , null)
.then(res => {
    log(res) // 资源文件
})
```

### EventManager (事件管理)

用于进行事件订阅和发布

#### EventManagerClass

该代码源文件位于 **根目录/Module/CcNative/EventManager/index.ts**

EventManagerClass 的构造定义为: 

```typescript
// 构造定义
class EventManagerClass {
    constructor() {}
}
```

它提供了一个全局的 instance 实例

```typescript
import { EventManagerClass } from "根目录/Module/EventManager"

const event = EventManagerClass.instance
// 或者
// const event = new EventManagerClass

// 回调函数
const callback = () => {}

// 监听 type name 事件
// type name 为事件名称 callback 为对于回调函数 1 就是执行次数(-1为无限执行，1为只执行一次)
event.on("type name" , callback , 1)

// 移除监听
event.off("type name" , callback)

// 提交事件 可以传入参数给callback
event.emit("type name" , ...arguments)

// 清楚事件所有回调
event.clear("type name")
```

### Extension (扩展模块)

针对于Cocos进行扩展操作，目前是两个常用组件进行的扩展

#### Component 

对组件进行扩展，一般是通用扩展

##### ExtensionComponent

该代码源文件位于 **根目录/Module/Extension/Component/ExtensionComponent.ts**

扩展基础  **cc.Component ** 添加了两个功能性的函数 **effect** 和 **setAutoInterval**

示例: 

```typescript
import { Extension } from '../../../../Module/Extension';

export default class MyComponent extends Extension.ExtensionComponent {
    
    protected start(): void {
        // 和 Vue3 中的响应式 effect 用法一样
        // 扩展了 effect 函数，会在组件销毁时自动停止
        this.effect(() => {})
        // 扩展了定时器函数，会在组件销毁时自动停止
        this.setAutoInterval(() => {} , {
            count: 1, // 执行次数 1 为 1 次 ， -1 为无限次
            timer: 1000 // 执行间隔 单位 ms
        })
    }
    
}
```

##### SpineAnimation

该代码源文件位于 **根目录/Module/Extension/Component/SpineAnimation.ts**

扩展基础  **cc.sp.Skeleton** 添加了一个功能性的函数 **playAnimation** 用于播放spine动画

示例: 

```typescript
import { Extension } from '../../../../Module/Extension';

export default class MyComponent extends Extension.SpineAnimation {
    
    protected start(): void {
        // playAnimation 有两个次数 动画名称 播放次数(-1为循环播放)
        this.playAnimation("动画名称" , 1)
        // 返回 Promise 如果动画有播放次数限制，则到达次数后 Promise 成功
        .then(res => {
            log("动画播放完成")
        })
    }
    
}
```

### Rx (响应式模块)

移植自 Vue3 核心响应式模块，主要有 **reactive** 和 **effect** 两个函数组成

具体用法和原理参考 [Vue3 响应式原理解析](https://mp.weixin.qq.com/s?__biz=MzI3NTM5NDgzOA==&mid=2247483736&idx=1&sn=7ffba2b40fa0ddacd1ad0a34c4afad7d&chksm=eb043921dc73b037b637f83a9344f9ffb40f0c514cce6cb2de609093d20860c436cd4d307db8&token=431470234&lang=zh_CN#rd)

基础用法为:

```typescript
const {Rx} from "根目录/Module/Rx"

// data 为可响应对象
const data = Rx.reactive({
    count: 0
})

Rx.effect(() => {
    // 使用自动订阅 data.count 的改变
    log(data.count)
})

// 改变 data.count
setInterval(() => data.count++ , 1500)

// 以上代码的执行结果为
// 每隔 1.5s 会输出 count 的值
```



## Script (主要脚本文件夹)

这里着重介绍 **Mod** 和 **System** 文件夹，扩展主要在 **Mod** 文件夹中，会经常用到 **System** 文件夹的类和函数以及各种注册装饰器。这里会从创建一个一个物品，装备，关卡，怪物，buff，技能来讲解...

### 自定义物品

游戏中的所有物品继承于最基础的物品原型类：ItemPrototype 类，它位于 **根目录/Script/System/Prototype/ItemPrototype.ts** 文件 ，我们通过重写基类的方法和属性可以做到自定义图标，介绍，用法等...

### 第一个自定义物品

假设我们现在有一个新的物品图标

![](./README/FlameEnhancementStone.png)

我们将它命名为 **君焰石** 作为一个物品，图片命名为 **FlameEnhancementStone.png**

我们先将它放入资源文件夹中 **根目录/Resource/Images/Item** 下

之后我们新创建脚本，将它创建在 **根目录/Script/Mod/Item** 下，就命名为 FlameEnhancementStone.ts

代码内容为: 

```typescript
// 从 System 文件夹中加载 ItemPrototype 基类 和 注册物品 的注解
import { ItemPrototype, RegisterItem } from "../../System/Prototype/ItemPrototype"

// 注册物品原型，参数为物品id，这里物品类就会被收录到 ItemPrototype.AllItems 中,可以被遍历使用
@RegisterItem("FlameEnhancementStone")
// 继承 ItemPrototype 类
export class FlameEnhancementStone extends ItemPrototype {

    // 重写名称属性，游戏内显示的名称
    public name: string = "君焰石"

    // 游戏内的图标路径
    public icon: string = "Images/Item/FlameEnhancementStone/spriteFrame"

    // 是否可以使用，如果可以的话，游戏内会显示使用按钮
    public canUse: boolean = false

    // 游戏中的简介
    public description: string = "蕴含着火焰力量的神秘石头"

    // 构造器
    constructor() {
        // 必须将自己的类传给父类
        super(FlameEnhancementStone)
    }

    // 使用时的函数
    public use(num: number): void {
    }

}
```

然后，我们就可以在游戏中获得物品了，但是我们现在没有获取路径，所以让我们再加入一个物品，用来获取所有被注册的物品。

### 第二个自定义物品

我们现在又有一个新的物品图标

![](./README/cheat-pack.png)

我们将它命名为 **作弊道具** 作为一个物品，图片命名为 **CheatPack.png**

我们先将它放入资源文件夹中 **根目录/Resource/Images/Item** 下

之后我们新创建脚本，将它创建在 **根目录/Script/Mod/Item** 下，就命名为 CheatPack.ts

代码内容为: 

```typescript
import { UserBagEquipments, UserEquipmentDTO } from "../../Data/UserBagEquipments"
import { ItemDto, UserBagItems } from "../../Data/UserBagItems"
import { EquipmentQuality } from "../../System/Instance/Equipment"
import { EquipmentPrototype } from "../../System/Prototype/EquipmentPrototype"
import { ItemPrototype, RegisterItem } from "../../System/Prototype/ItemPrototype"
import { MessageAlert } from "../../Util/MessageAlert"

// 注册物品
@RegisterItem("CheatPack")
export class CheatPack extends ItemPrototype {

    public name: string = "开发者作弊礼包"

    public icon: string = "Images/Item/CheatPack/spriteFrame"

    public canUse: boolean = true

    public description: string = "获得所有装备和物品"

    constructor() { 
        super(CheatPack)
    }

    // num 为 使用数量
    public use(num: number): void {
        // 获取所有注册的物品id
        const allItemConstroctor = Array.from(ItemPrototype.AllItems.keys())
        // 物品数据 ItemDto {id: string , count: number}
        let items: ItemDto[] = []
        // 循环使用
        for (let i = 0; i < num; i++) {
            // 遍历所有物品
            allItemConstroctor.forEach((key) => {
                // 避免重复添加本身物品
                if (key !== "CheatPack") {
                    // 通过 UserBagItems.addItem 函数将物品添加到用户数据中
                    const result = UserBagItems.addItem({id: key, count: 1000})
                    // 添加物品数据
                    items.push(result)
                }
            })
        }
        // 展示祝贺语
        MessageAlert.congratulationsGetRewards(items , [])
        // 保存用户数据到本地
        UserBagItems.saveInstance()
    }

}
```

### 通过代码添加物品到背包

现在我们将该物品添加到用户初始物品栏中，来到文件 **根目录/Script/Data/UserBagItems.ts** 文件

在代码中添加

```typescript
export class UserBagItems {

    // ... 其余代码
    
    // 修改 items 赋予初始值 ， id为我们使用 RegisterItem 装饰器设置的id ，count 为数量
    public items: Array<ItemDto> = [{id: "CheatPack" , count: 99}];

}
```

至此，我们就可以在游戏中看到

![](./README/04.png)

物品就出现了。**如果您没有找到对应的物品，可能是您之前进入游戏已经保存过了，只需要清除浏览器缓存(locaStorage)重新进入游戏即可!!!**

这里我们就可以使用物品了，使用物品后，这个物品会遍历所有物品并且添加到背包中并弹出提示，之后就可以看到我们的君焰石了

![](./README/06.png)

至此，物品的添加就先告一段落了，你已经成功添加了两个自己的物品到你的游戏中了，接下来我们试试添加一个技能到游戏中吧。

### 添加一个自己的技能

游戏中的所有技能继承于最基础的技能原型类：SkillPrototype 类，它位于 **根目录/Script/System/Prototype/SkillPrototype .ts** 文件 ，我们通过重写基类的方法和属性可以做到自定义图标，介绍，用法等...

### 第一个自定义技能

假设我们现在有一个新的技能图标

![](./README/PlayerSkillMacrotherapy.png)

我们将它命名为 **大治疗术** 作为一个物品，图片命名为 **Macrotherapy.png**

我们先将它放入资源文件夹中 **根目录/Resource/Images/Skill** 下

之后我们新创建脚本，将它创建在 **根目录/Script/Mod/Skill** 下，就命名为 Macrotherapy.ts

代码内容为

```typescript
import { RegisterSkill, SkillPrototype } from '../../System/Prototype/SkillPrototype';
import { RegisterPlayerSkill } from '../../Data/UserSkillTree';
import { FightData } from '../../System/Base/FightData';

// 注册技能
@RegisterSkill("Macrotherapy")
// 注册到玩家技能列表
@RegisterPlayerSkill("Macrotherapy")
export class Macrotherapy extends SkillPrototype {

    // 游戏内显示的名称
    public name: string = "大治疗术"

    // 游戏内显示的图标
    public icon: string = "Images/Skill/Macrotherapy/spriteFrame"

    // 简介
    public get description(): string {
        return `治疗所有队友，恢复 ${
            this.getCure()
        }(${
            100 * this.getLv()
        }%魔力值 + ${
            50 * this.getLv()
        }) 的生命值 , 并获得 ${ tihs.getDefense() } 的双抗 持续5s`
    }

    // 冷却时间，单位为秒
    public get time(): number {
        return 10 - Math.min(this.getLv() , 5)
    }
    
    // 消耗魔力值
    public get cost(): number {
        return 20 + (10 * this.getLv())
    }

    // 获取规范化等级
    protected getLv() {
        return Math.max(1 , this.skill.lv)
    }

    // 获取治疗量，治疗量 由 魔力 角色技能等级 决定
    protected getCure() {
        return this.skill.character.magic * Math.max(1 , this.getLv()) + 50 * this.getLv()
    }
    
    // 获取双抗
    protected getDefense() {
        return 10 * this.getLv()
    }
    
    // 使用回调
    public use(fightData: FightData): void {
        // 如果 FightData 中的 player 属性为 当前技能的所属角色，则该技能为玩家使用
        if (fightData.player === this.skill.character) {
            // 只治疗玩家 heal 函数为 Character 类的方法，用于治疗角色
            fightData.player.heal(
                fightData.player , // 治疗来自谁
                this.getCure() // 治疗量
            )
            // 添加buff
        }
        // 否则为怪物使用
        else {
            // 遍历所有怪物
            fightData.monsters.forEach(monster => {
                // 如果怪物位为空或者怪物已经死亡
                if (!monster || monster.isDead) return
                // 治疗怪物
                monster.heal(this.skill.character , this.getCure())
                // 添加buff
            })
        }
    }

    // 必须重写构造器，并且传入当前类给父类
    constructor() {
        super(Macrotherapy)
    }

}
```

这里我们先不添加 Buff ， 之后我们创建自定义 Buff 后再回来给角色添加对应的 Buff。之后我们就可以在技能界面查看我们新添加的技能了

![](./README/07.png)

之后就可以学习并且装配使用对应的技能了，接下来，我们去实现一个自定义的 Buff 来完善该技能增加双抗并且持续5s的功能。
