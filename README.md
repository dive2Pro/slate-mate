# slate-mate
[Slate](https://github.com/ianstormtaylor/slate) medium-like editor with exposed decorators and plugins.
Large parts of the code is based on @ianstormtaylor's excellent examples, so props to him.
Also, it's been a breeze to work with slate, since the API is extremely well thought through. Give it a try!

The purpose of this library is to provide a ready-to-use editor, as well as easy to use decorators for slate and atomic blocks, to help you get started as easily as possible!
**Documentation not quite ready yet!**

## Editor decorators
Most decorators will get options through decorator initialization arguments AND through the props.
- [state](https://github.com/bkniffler/slate-mate/blob/master/src/editor-decorators/state.js) (accept a raw json value, manages state, calls onChange)
- [block](https://github.com/bkniffler/slate-mate/blob/master/src/editor-decorators/blocks.js) (adds empty lines after blocks, accept 'blockTypes'-object through props and use it to construct schema nodes and sidebar items)
- [toolbar](https://github.com/bkniffler/slate-mate/blob/master/src/editor-decorators/toolbar.js) (medium like inline toolbar)
- [sidebar](https://github.com/bkniffler/slate-mate/blob/master/src/editor-decorators/sidebar.js) (+ on the side of empty lines to add atomic blocks)
- [auto-markdown](https://github.com/bkniffler/slate-mate/blob/master/src/editor-decorators/auto-markdown.js) (initialize lists with '-' or titles with '#')

## Block decorators
- [base](https://github.com/bkniffler/slate-mate/blob/master/src/block-decorators/base.js) (exposes `setData` and `getData` props for easy block data and `isFocused` prop)
- [toolbar](https://github.com/bkniffler/slate-mate/blob/master/src/block-decorators/toolbar.js) (render a block toolbar from actions, either automatically or manually via `manual: true` option and `setToolbarPosition`)
- [align](https://github.com/bkniffler/slate-mate/blob/master/src/block-decorators/align.js) (set alignment toolbar-actions, provide alignment styles, expose `setAlignment` and `alignment` prop)
- [resize](https://github.com/bkniffler/slate-mate/blob/master/src/block-decorators/resize.js) (make a block resizable, either with aspect ratio via `ratio` option or freely)

## Example editor
- [Editor](https://github.com/bkniffler/slate-mate/blob/master/docs/editor.js)
```jsx
import { Editor, Mark } from 'slate';
import React, { Component, PropTypes } from 'react';
import { withState, withSidebar, withToolbar, withAutoMarkdown, useBlocks } from 'slate-mate';

const options = {
  defaultNode: 'paragraph',
  blockTypes: {
    'youtube-block': YoutubeBlock,
  },
  toolbarMarks: [
    { type: 'bold', icon: 'bold' },
    { type: 'italic', icon: 'italic' },
    { type: 'underlined', icon: 'underline' },
    { type: 'code', icon: 'code' },
  ],
  toolbarTypes: [
    { type: 'heading-one', icon: 'header' },
    { type: 'heading-two', icon: 'header' },
    { type: 'block-quote', icon: 'quote-left' },
    { type: 'numbered-list', icon: 'list-ol' },
    { type: 'bulleted-list', icon: 'list-ul' },
  ],
  sidebarTypes: [],
  nodes: {
    'block-quote': ({ children }) => <blockquote>{children}</blockquote>,
    'bulleted-list': ({ children }) => <ul>{children}</ul>,
    'numbered-list': ({ children, attributes }) => <ol {...attributes}>{children}</ol>,
    'heading-one': ({ children }) => <h1>{children}</h1>,
    'heading-two': ({ children }) => <h2>{children}</h2>,
    'heading-three': ({ children }) => <h3>{children}</h3>,
    'heading-four': ({ children }) => <h4>{children}</h4>,
    'heading-five': ({ children }) => <h5>{children}</h5>,
    'heading-six': ({ children }) => <h6>{children}</h6>,
    'bulleted-list-item': ({ children }) => <li>{children}</li>,
    'numbered-list-item': ({ children }) => <li>{children}</li>,
  },
  marks: {
    bold: ({ children }) => <strong>{children}</strong>,
    code: ({ children }) => <code>{children}</code>,
    italic: ({ children }) => <em>{children}</em>,
    underlined: ({ children }) => <u>{children}</u>,
  },
  getMarkdownType: (chars) => {
    switch (chars) {
      case '*':
      case '-':
      case '+': return 'bulleted-list-item';
      case '>': return 'block-quote';
      case '#': return 'heading-one';
      case '##': return 'heading-two';
      case '###': return 'heading-three';
      case '####': return 'heading-four';
      case '#####': return 'heading-five';
      case '######': return 'heading-six';
      case '1.': return 'numbered-list-item';
      default: return null;
    }
  },
};

@withState()
@useBlocks(options)
@withAutoMarkdown(options)
@withToolbar(options)
@withSidebar(options)
export default class SlateEditor extends Component {
  static propTypes = {
    readOnly: PropTypes.bool,
    children: PropTypes.node,
    value: PropTypes.object,
    onChange: PropTypes.func,
    marks: PropTypes.object,
    nodes: PropTypes.object,
    autoMarkDownKeyDown: PropTypes.func,
    plugins: PropTypes.array,
  }
  render = () => {
    const { children, value, onChange, readOnly, marks, nodes, plugins } = this.props;
    return (
      <div className="editor">
        {children}
        <Editor
          readOnly={readOnly}
          plugins={plugins}
          schema={{ marks, nodes }}
          state={value}
          onChange={onChange}
        />
      </div>
    );
  }
}
```

## Example Youtube block
- [youtube-block](https://github.com/bkniffler/slate-mate/blob/master/docs/youtube-block.js)
```jsx
import React, { Component, PropTypes } from 'react';
import { useBlockBase, useBlockResize, useBlockAlign, useBlockToolbar } from 'slate-mate';

const defaultVideo = 'https://www.youtube.com/embed/zalYJacOhpo';
const actions = props => [{
  type: 'youtube.url',
  icon: 'picture-o',
  toggle: () => {
    const { setData, getData } = props;
    const currentUrl = getData('url') || defaultVideo;
    const url = window.prompt('URL', currentUrl);
    if (url) setData({ url });
  },
  active: false,
}];

@useBlockBase()
@useBlockAlign()
@useBlockResize({ ratio: 7 / 4 })
@useBlockToolbar({ actions })
export default class YoutubeBlock extends Component {
  static propTypes = {
    children: PropTypes.node,
    style: PropTypes.object,
    className: PropTypes.string,
    isFocused: PropTypes.bool,
    attributes: PropTypes.object,
    getData: PropTypes.func,
  }
  static title = 'Youtube';
  static icon = 'youtube';
  static category = 'Media';

  render() {
    const { style, getData, className, children, isFocused, attributes } = this.props;
    const url = getData('url', defaultVideo);

    const styles = {
      backgroundColor: 'gray',
      width: '100%',
      height: '100%',
      position: 'relative',
      ...style,
    };

    return (
      <div {...attributes} style={styles} className={className} data-block-active={isFocused}>
        <iframe width="100%" height="100%" src={url} frameBorder="0" allowFullScreen />
        {children}
      </div>
    );
  }
}
```
