# l3buld-glob-issue

I'm having an issue where typesetfiles in a subdirectory get compiled, but then moved to a strangely named file in the current directory.
This repository demonstrates my issue.

```
.
├── build.lua
├── examples
│   ├── bar.tex
│   └── foo.tex
├── simple-flat.dtx
└── simple-flat.ins

2 directories, 5 files
```

The contents of `build.lua` are

```lua
module = "l3build-glob-issue"

typesetfiles = { "*.dtx", "examples/*.tex"}
```

And `foo.tex` and `bar.tex` are arbitrary TeX files.

When I execute `l3build doc`, I get `simple-flat.pdf` in the main directory. But also:

* the file `example/bar.tex` is copied into `build/doc/`
* the file `bar.tex` is compiled in `build/doc`
* the product `bar.pdf` is copied to a file in the main directory called  **`.xamples`**
* the same three steps are repeated for `example/foo.tex`.

If you open `.xamples` in a PDF viewer, you will see it's a copy of `foo.pdf`.

Copilot tells me the problem is caused by [this part of l3build-typesetting.lua](https://github.com/search?q=repo%3Alatex3%2Fl3build%20destpath%20%3D%20docfiledir%20..%20gsub(gsub(destpath%2C%22%5E.%2F%22%2C%22%22)%2C%22%5E.%22%2C%22%22)&type=code):

> During doc output placement, l3build computes a destination path from the glob directory part and applies a pattern that removes the first character of that directory name. So `examples` becomes `xamples`, then with the default `docfiledir` of `.` it becomes `.xamples`.

So I guess `typesetfiles` isn't equipped to handle globs in subdirectories. Am I configuring this wrong? 

The workaround I'm using now is copying the `example/*.tex` files into `typesetdir` inside of `docinit_hook`, then changing `"example/*.tex"` to `"*.tex"` in `typesetfiles`.
