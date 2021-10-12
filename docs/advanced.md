## Custom block

What is a custom block？Custom block is composed of one or more basic blocks.

This is a Section block with its children

```tsx
<Section>
  <Column>
    <Text>hello</Text>
  </Column>
</Section>
```

But we can also encapsulate it and call it Custom Section block.

```tsx
(<CustomSection></CustomSection>).isEqual(
  <Section>
    <Column>
      <Text>hello</Text>
    </Column>
  </Section>
);
```

There is such a conversion rule

`IBlockData<T>` => `transformToMjml`=> `mjml-component<T>`

- transformToMjml(`IText`) === `<mj-text>xxx</mj-text>`
- transformToMjml(`ISection`) === `<mj-section>xxx</mj-section>`

And it can be reversed

- `<mj-text>xxx</mj-text>` => `MjmlToJson` => `IText`

To use a custom block, there should be the following steps
- Write a custom block
- Register this block
- If you want to show it to the left panel, add it to `extraBlocks`

### Write a custom block

A custom block should have the following structure

```ts
{
  name: string; // block name, show in tooltip
  type: BlockType; // Custom type
  Panel: () => React.ReactNode; // Configuration panel, update your block data
  validParentType: BlockType[]; // Only drag to the above blocks. For example, `Text` only drag to `Colum` block and `Hero` block.
  create: (payload?: RecursivePartial<T extends IBlockData>) => T;
  render?: (
    data: IBlockData<T>, // current block data
    idx: string, // current idx
    context: IPage //
  ) => IBlockData;
}

```

`create` is a method of instance generation, Let’s say `Text`, when dragging and dropped into the edit panel and , we will call `addBlock`. In fact, it just calls the corresponding `create` and generate blockData.

```ts
const create: CreateInstance<IText> = (payload) => {
  const defaultData: IText = {
    type: BasicType.TEXT,
    data: {
      value: {
        content: "Make it easy for everyone to compose emails!",
      },
    },
    attributes: {
      "font-size": "13px",
      padding: "10px 25px 10px 25px",
      "line-height": 1,
      align: "left",
    },
    children: [],
  };
  return merge(defaultData, payload);
};
```

`render` mainly to render your custom block into one or more basic block. When transformToMjml is called, if it is found to be an custom block, we will call its `render` method to convert it into basic blocks. At the same time, the following parameters will be injected `currentBlockData, current idx, page context`.

You can construct your custom block through basic blocks. For example,
a custom button, only the background color and text can be modified

```tsx
import { Button } from "easy-email-editor";

const render = (
  data: ICustomButton,
  idx: string,
  context: IPage
): IBlockData => {
  const attributes = data.attributes;
  const { buttonText } = data.data.value;

  const instance = (
    <Button background-color={attributes["background-color"]}>
      {buttonText}
    </Button>
  );

  return instance;
};
```

Another way is that you can write [MJML](https://documentation.mjml.io/).

```ts
import { MjmlToJson } from 'easy-email-editor';

const render = (data: ICustomButton, idx:string; context: IPage): IBlockData => {
  const attributes = data.attributes;
  const { buttonText } = data.data.value;

  const instance = MjmlToJson(`<mj-button background-color=${attributes['background-color']}>${buttonText}</mj-button>`)

  return instance;
};

```

### Register this block
Only after registering this block, mjml-parser can convert it into basic blocks

```ts
import { BlocksMap } from "easy-email-editor";

BlocksMap.registerBlocks({ "block-name": YourCustomBlock });
```

### Show it to the left side


```js
import { BlocksMap } from "easy-email-editor";
import { ProductRecommendation } from "./ProductRecommendation";
import { CustomBlocksType } from './constants';
import { Example as YourCustomBlockExamples } from "./ProductRecommendation/Example";

export const customBlocks = {
  title: "category title",
  name: "category name",
  blocks: [
     {
      type: CustomBlocksType.PRODUCT_RECOMMENDATION as any,
      title: ProductRecommendation.name,
      ExampleComponent: Example, 
      thumbnail:
        'https://assets.maocanhua.cn/c160738b-db01-4081-89e5-e35bd3a34470-image.png', 
    },
  ],
};
```
<img width="600" src="https://assets.maocanhua.cn/5d6f4763-f69b-48ec-8bb4-99101c4b8805-image.png" />

### View demo
 [https://github.com/m-Ryan/easy-email-demo/tree/main/src/CustomBlocks](https://github.com/m-Ryan/easy-email-demo/tree/main/src/CustomBlocks)

<br/>
<br/>

## Dynamic rendering

### Basic block 
you can defined `merge tags`, and replace it before sending. We recommend [mustache](https://github.com/janl/mustache.js)

```ts

import Mustache from 'mustache';

const mergeTag = {
  name: "ryan",
  date: function () {
    return new Date().toLocaleDateString();
  }
};

function injectData(data: IEmailTemplate['content']) {
    // data have a text with content => "Hello {{name}}, today is {{date}}"
    return JSON.parse(Mustache.render(JSON.stringify(data), mergeTag));
}



```

### Custom block
you can defined your block's render logic. You just need to replace its data before sending.

```js

function injectData(data: IEmailTemplate['content']) {
  const testData = cloneDeep(data);
  testData.children.forEach((item) => {
    if (item.type === CustomBlocksType.PRODUCT_RECOMMENDATION) {
      item.data.value.productList = [
          {
            image:
              'https://assets.maocanhua.cn/da9b173d-b272-4101-aa25-4635ed95e9e3-image.png',
            title: 'Slim Fit Printed shirt',
            price: '$59.99 HKD',
            url: 'https://easy-email-m-ryan.vercel.app',
          },
          {
            image:
              'https://assets.maocanhua.cn/4ef7cb65-ee1f-4b12-832c-17ab07a8b9ac-image.png',
            title: 'Casual Collar Youth Handsome Slim Print Blazer',
            price: '$59.99 HKD',
            url: 'https://easy-email-m-ryan.vercel.app',
          },
          {
            image:
              'https://assets.maocanhua.cn/88fe9bfa-547f-4d5e-9ba5-ac6b91572dde-image.png',
            title: 'Shirt Business Casual',
            price: '$59.99 HKD',
            url: 'https://easy-email-m-ryan.vercel.app',
          },
        ]
    }
  });

  return testData;
}

```




