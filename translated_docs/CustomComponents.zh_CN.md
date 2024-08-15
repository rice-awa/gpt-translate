请将以下文本翻译成自然的中文。
---
author: mammerla
ms.author: mikeam
title: '介绍自定义组件'
description: "介绍一种新的实验性概念——自定义组件。"
ms.service: minecraft-bedrock-edition
---

# 介绍自定义组件

自定义组件是一种将方块和物品的JSON配置与脚本功能直接且有针对性地连接的新方式。这一新概念允许在方块和物品之间组合和重用脚本功能，同时确保脚本仅针对特定的方块和物品运行。

这种新模式结合了一种更结构化的方式来监听方块和物品事件，同时获取脚本中的所有功能，并确保这些事件在与当前脚本事件相同的约束下运行。

如果你想开始使用自定义组件构建附加组件，请参阅我们的[自定义组件教程](./CustomComponentsTutorial.md)。

## 结构

这个新功能被命名为_自定义组件_，因为脚本以一种非常类似于附加内置Minecraft组件的方式连接到给定的方块或物品的JSON。因此，该功能既有JSON方面的内容，也有脚本方面的内容。

## 脚本API

从脚本API开始，我们为方块引入了两个新的主要接口：`BlockTypeRegistry`和`BlockCustomComponent`。（注意：在1.21.0之后的Minecraft预览版本中，BlockTypeRegistry被称为BlockComponentRegistry）`BlockTypeRegistry`包含一个用于按名称注册新自定义组件的方法：

```typescript
/**
 * @beta
 * 提供注册方块自定义组件的功能。
 */
export class BlockTypeRegistry {
    /**
     * @remarks
     * 注册一个可以在方块JSON配置中使用的方块自定义组件。
     *
     * @param name
     * 表示此自定义组件的ID。必须有一个命名空间。此ID可以在方块的JSON配置中的'minecraft:custom_components'方块组件下指定。
     * @param customComponent
     * 当使用此自定义组件ID的方块上发生事件时，将调用的事件函数集合。
     */
    registerCustomComponent(name: string, customComponent: BlockCustomComponent): void;
}
```

脚本中的自定义组件是名称/ID与一组由事件表示的功能的关联。可以监听的事件在第二个接口`BlockCustomComponent`中指定。

```typescript
/**
 * @beta
 * 包含将为方块引发的一组事件。
 * 此对象必须使用BlockRegistry绑定。
 */
export interface BlockCustomComponent {
    /**
     * @remarks
     * 当实体踩到绑定了此自定义组件的方块上时，将调用此函数。
     *
     */
    onStepOn?: (arg: BlockComponentStepOnEvent) => void;
}
```

在脚本中，你可以通过监听新的实验性`worldInitialize` _before_ 事件来访问`BlockTypeRegistry`。所有自定义组件的注册必须在worldInitialize期间进行，因为此功能直接附加到JSON中的方块初始化。在此事件中，你可以调用`registerCustomComponent`，使用唯一的命名空间名称和实现`BlockCustomComponent`接口的对象来注册该组件。一旦组件注册，任何使用此组件的方块都会在相关事件中调用你的对象上的回调。

以下是一个显示此注册的小代码示例：

```typescript
world.beforeEvents.worldInitialize.subscribe(initEvent => {
    initEvent.blockTypeRegistry.registerCustomComponent('content:turn_to_air', {
        onStepOn: e => {
            e.block.setPermutation(BlockPermutation.resolve(MinecraftBlockTypes.Air));
        },
    });
});
```

在上面的示例中，注册了一个名为`content:turn_to_air`的自定义组件，该组件监听`onStepOn`事件。
也可以将JavaScript类注册为自定义组件；唯一重要的是遵循`BlockCustomComponent`接口。以下是使用类模式的示例。

```typescript
class TurnToAirComponent implements BlockCustomComponent {
    constructor() {
        this.onStepOn = this.onStepOn.bind(this);
    }

    onStepOn(e: BlockComponentStepOnEvent): void {
        e.block.setPermutation(BlockPermutation.resolve(MinecraftBlockTypes.Air));
    }
}

world.beforeEvents.worldInitialize.subscribe(initEvent => {
    initEvent.blockTypeRegistry.registerCustomComponent('content:turn_to_air', new TurnToAirComponent());
});
```

根据你的附加组件或世界的需求，你可以选择你喜欢的模式！它们在功能上是相同的。需要注意的是，所有事件通知都是无状态的，事件参数会告知你它们在世界中对应的具体方块。

## JSON

注册到特定名称的自定义组件后，接下来就是如何将其附加到自定义方块的问题。
在自定义方块的JSON中，在`components`键下，有一个新的`minecraft:custom_components`键，可以在其中注册自定义组件。自定义组件在此处按顺序数组列出。此数组让你可以控制在特定事件中通知注册的自定义组件的顺序。以下是注册我们的`content:turn_to_air`自定义组件的示例。

```JSON
    "minecraft:block": {
        "description": {
            "identifier": "content:my_block"
        },
        // 基础组件
        "components": {
            "minecraft:custom_components": ["content:turn_to_air"],
            "minecraft:loot": "loot_tables/blocks/my_block.json",
```

在上面的示例中，自定义组件与其他组件（如战利品表）一起附加。如果注册了多个自定义组件，那么对于每种类型的事件，事件将按此数组中的顺序发送到自定义组件。

也可以在`components`键下的每个特定排列中添加自定义组件。然而，需要注意的是，排列中的自定义组件数组完全替换了基础组件列表中的数组。
同一个自定义组件也可以在多个文件中使用。例如，这个`content:turn_to_air`组件可以在多个自定义方块中重用，以实现轻松对齐和组合的行为。此外，一个行为包中注册的自定义组件可以在另一个行为包中使用。

## 工作流程

在编写自定义组件及其相应的脚本代码时，你可以充分利用热重载的功能，在世界中迭代你的更改。

当更改JSON和/或注册新的自定义组件时，需要退出世界并重新进入，以查看你的更改。

一旦进入脚本，你的回调可以像今天的任何其他脚本一样使用脚本API中的任何API，并具有相同的约束。

## 开始构建自定义组件

如果你想开始使用自定义组件构建附加组件，请访问[自定义组件教程](./CustomComponentsTutorial.md)。需要注意的是，在1.21.0之后的Minecraft预览版本中，BlockTypeRegistry被称为BlockComponentRegistry。