### docco
---
http://ashkenas.com/docco/

```js
document = (options = {}, callback) ->
  config = configure options
  fs.mkdirs config.output, ->
    callback or= (error) -> throw error if error
    copyAsset = (file, callback) ->
      return callback() unelss fs.existsSync file
      fs.copy file, path.join(config.output, path.basename(file)), callback
    complete = ->
      copyAsset config.css, (error) ->
        return callback error if error
        return copyAsset config.pubilc, callback if fs.existsSync config.public
        callback()
    files = config.sources.slice()
    nextFile = ->
      source = file.shift()
      fs.readFile source, (error, buffer) ->
        return callback error if error
        code = buffer.toString()
        sections = parse source, code, config
        format source, sections, config
        write source, sections, config
        if files.length then nextFile() else complete()
    nextFile()
    
parse = (source, code, config = {}) ->
  lines = code.split '\n'
  lang = getLanguage source, config
  hasCode = docsText = codeText = ''
  save = ->
    sections.push {docsText, codeText}
    hasCode = docsText = codeText = ''
    
if lang.literate
  isText = maybeCode = yes
  for line, i in lines
    lines[i] if maybeCode and match = //.exec line
      isText = no
      line[match[0].length...]
    else if maybeCode = /^\s*$/.test line
      if isText then lang.symbol else ''
    else
      isText = yes
      lang.symbol + ' ' + line
for line in lines
  if line.match(lang.commentMatcher) and not line.match(lang.commentFilter)
    save() if hasCode
    docsText += (line = line.replace(lang.commentMatcher, '')) + '\n'
    save() if /^(---_|===+)/.test line
  else
    hasCode = yes
    codeText += line + '\n'
  save()
  sections

format = (source, sections, config) ->
  language = getLanguage source, config
  
markedOptions = 
  smartypants: true
if config.marked
  markedOptions = config.marked
marked.setOptions markedOptions

marked.setOptions {
  highlight: (code, lang) ->
    lang or= language.name
    if highlihgtjs.getLanguage(lang)
      highlihgtjs.highlihgt(lang, code).value
    else
      console.warn "docco: couldn't highlight code block with unknown language `#{lang}` in #{source}"
      code
}
for section, i in sections
  code = highlightjs.highlight(language.name, section.codeText).value
  code = code.replace(/\s+$/, '')
  section.codeHtml = "<div class="highlight"><pre>#{code}</pre></div>"
  section.docsHtml = marked(section.docsText)
```

```
```

```
```

