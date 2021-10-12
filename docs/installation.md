# Installation

```bash
npm install --save easy-email-editor antd mjml-browser react-final-form

```

or

```bash

yarn add easy-email-editor antd mjml-browser react-final-form

```

# Usage:

```js
import React from "react";
import { BlocksMap, EmailEditor, EmailEditorProvider } from "easy-email-editor";
import "easy-email-editor/lib/style.css";
import "antd/dist/antd.css";

const initialValues = {
  subject: "Welcome to Easy-email",
  subTitle: "Nice to meet you!",
  content: BlocksMap.getBlock("Page").create({}),
};

export function App() {
  return (
    <EmailEditorProvider autocomplete data={initialValues}>
      {({ values }) => {
        return <EmailEditor height={"calc(100vh - 72px)"} />;
      }}
    </EmailEditorProvider>
  );
}
```

# Configuration

- ## EmailEditorProvider

  | property           | Type                                                                                               | Description                                                                                         |
  | ------------------ | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
  | data               | interface IEmailTemplate { content: IPage; subject: string; subTitle: string; }                    | Source data                                                                                         |
  | children           | ( props: FormState<T>,helper: FormApi<IEmailTemplate, Partial<IEmailTemplate>>) => React.ReactNode | ReactNode                                                                                           |
  | onSubmit           | Config<IEmailTemplate, Partial<IEmailTemplate>>['onSubmit'];                                       | Called when the commit is triggered manually                                                        |
  | fontList           | { value: string; label: string; }[];                                                               | Default font list.                                                                                  |
  | interactiveStyle   | { hoverColor?: string; selectedColor?: string;}                                                    | Interactive prompt color                                                                            |
  | onUploadImage      | (data: Blob) => Promise<string>;                                                                   | Triggered when an image is pasted or uploaded                                                       |
  | onAddCollection    | (payload: CollectedBlock) => void;                                                                 | Add to collection list                                                                              |
  | onRemoveCollection | (payload: { id: string; }) => void;                                                                | Remove from collection list                                                                         |
  | dashed             | boolean                                                                                            | Show dashed                                                                                         |
  | autoComplete       | boolean                                                                                            | Automatically complete missing blocks. For example, Text => Section, will generate Text=>Column=>Section |

- ## EmailEditor

  | property | Type            | Description                     |
  | -------- | --------------- | ------------------------------- |
  | height   | string / number | Set the height of the container |
