<template>
  <div id="app">
    <header class="header">
      <div class="container">
        <div class="header-logo" style="background-image: url(https://github.com/vueact.png)">
        </div>
        <div class="header-left">

          <h1>

          <a href="https://github.com/vueact/babel-plugin-transform-react-to-vue">babel-plugin-transform-react-to-vue</a> REPL</h1>
          <h2>Built using <a href="https://github.com/egoist/poi">Poi</a> with almost zero-config, <a target="_blank" href="https://github.com/vueact/plugin-repl">check out repl source code</a></h2>
        </div>

      </div>
    </header>
    <div class="editors">
      <div class="container">
        <textarea ref="input" class="input">{{ code }}</textarea>
        <div class="result">
          <pre class="code cm-s-oceanic"><code v-html="result"></code></pre>
          <div class="error" v-show="error">{{ error }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import highlight from 'cm-highlight'
  import CodeMirror from 'codemirror'
  import qs from 'querystring'

  import 'codemirror/mode/javascript/javascript'
  import 'codemirror/mode/jsx/jsx'
  import 'codemirror/addon/selection/active-line'
  import 'codemirror/addon/fold/xml-fold'
  import 'codemirror/addon/edit/matchtags'

  export default {
    name: 'JSXEditor',
    data() {
      const defaultValue = `
import ReactDOM from 'react-dom'
import React, { Component } from 'react'

class App extends Component {
  constructor(props) {
    super(props)
    this.state = { hello: 'world' }
  }
  myMethod = () => {
    this.setState({ hello: 'not world ;)' })
  }
  render() {
    return (
      <div className="App">
        <div className="App-header" onClick={this.myMethod}>
          <h2>
            Hello {this.state.hello}
          </h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    )
  }
  componentDidMount = () => console.log(this.state)
}

ReactDOM.render(<App />, document.getElementById('root'))
`.trim()
      const { input } = qs.parse(location.hash.substring(2))

      return {
        result: 'Loading...',
        error: '',
        code: input || defaultValue
      }
    },
    mounted() {
      this.editor = CodeMirror.fromTextArea(this.$refs.input, {
        mode: 'jsx',
        tabSize: 2,
        indentWithTabs: false,
        theme: 'oceanic',
        styleActiveLine: true,
        matchTags: { bothTags: true },
        extraKeys: {
          Tab: cm => {
            cm.replaceSelection(' '.repeat(cm.getOption('tabSize')))
          }
        }
      })
      this.transform()
      this.editor.on('change', e => {
        this.code = e.getValue()
      })
    },
    watch: {
      code() {
        this.transform()
      },
      mode() {
        this.transform()
      }
    },
    methods: {
      async transform() {
        const code = this.code
        this.updateURL({ input: code })
        try {
          const [babel, preset] = await Promise.all([
            import('babel-standalone'),
            import('./preset').then(m => m.default)
          ])

          const transformOptions = {
            presets: [preset]
          }
          const result = babel.transform(code, transformOptions)

          this.result = highlight(result.code, {
            mode: 'jsx'
          })

          this.error = null
        } catch (err) {
          this.error = err.stack
        }
      },
      updateURL({ input }) {
        const query = qs.parse(location.hash.substring(2))
        if (input !== undefined) {
          query.input = input
        }
        location.hash = `#?${qs.stringify(query)}`
      }
    }
  }
</script>

<style src="codemirror/lib/codemirror.css"></style>
<style src="./codemirror.css"></style>
<style src="./codemirror-oceanic.css"></style>
<style>
  :root {
    --dark: #1C2022;
    --light: rgba(255,255,255,0.7);
  }
  html, body, #app, .CodeMirror {
    height: 100%;
  }
  body {
    margin: 0;
    background-color: var(--dark);
    font: 14px/1.4 -apple-system,BlinkMacSystemFont,Segoe UI,Roboto,Oxygen,Ubuntu,Cantarell,Fira Sans,Helvetica Neue,sans-serif;
  }
  * {
    box-sizing: border-box;
  }
  a {
    color: var(--light);
    text-decoration: none;
    &:hover {
      text-decoration: underline;
    }
  }
  .container {
    max-width: 1080px;
    margin: 0 auto;
    height: 100%;
  }
  .header {
    height: 80px;
    background-color: var(--dark);
    color: var(--light);
    >.container {
      display: flex;
      align-items: center;
    }
    h1 {
      margin: 0;
      font-weight: 400;
    }
    h2 {
      margin: 0;
      font-weight: 400;
      font-size: 14px;
    }
    .header-logo {
      width: 60px;
      height: 100%;
      background-size: contain;
      background-repeat: no-repeat;
      background-position: center;
      margin-right: 10px;
    }
    .header-right {
      display: flex;
      align-items: center;
    }
  }
  .editors {
    height: calc(100% - 80px);
    >.container {
      display: flex;
      align-items: center;
      justify-content: center;
      border: 1px solid rgb(18, 21, 22);
      border-radius: 3px 3px 0 0;
      overflow: hidden;
    }
    .CodeMirror, .result {
      width: 50%;
    }
  }
  .input {
    border: none;
    outline: none;
    resize: none;
    font-size: 1rem;
  }
  .result {
    position: relative;
    height: 100%;
    border-left: 1px solid #19161b;
  }
  .code {
    padding: 4px;
    margin: 0;
    height: 100%;
    word-wrap: normal;
    overflow: auto;
    line-height: 1.45;
    code {
      font-family: inconsolata, monospace;
      padding: 0;
      margin: 0;
      font-size: 100%;
      word-break: normal;
      white-space: pre;
      background: transparent;
      border: 0;
    }
  }
  .error {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: red;
    color: white;
    padding: 0;
    overflow: auto;
    padding: 10px;
    font-size: 16px;
    white-space: pre;
  }
</style>
