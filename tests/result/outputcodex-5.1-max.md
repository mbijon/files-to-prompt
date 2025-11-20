./AGENTS.md
```
 1  # Repository Guidelines
 2  
 3  ## Project Structure & Modules
 4  - Core package lives in `files_to_prompt/`: `cli.py` exposes the Click-based CLI, `__main__.py` enables `python -m files_to_prompt`, and `__init__.py` holds package metadata.
 5  - Tests sit in `tests/test_files_to_prompt.py`; keep new tests in this suite to mirror CLI behaviors.
 6  - Tooling and metadata are defined in `pyproject.toml`. Runtime dependency footprint is minimal (`click` only); avoid introducing new ones without discussion.
 7  
 8  ## Setup, Build, and Development Commands
 9  - Create an isolated environment: `python -m venv venv && source venv/bin/activate`.
10  - Install dependencies for local work: `pip install -e '.[test]'`.
11  - Run the CLI during development without installing globally: `python -m files_to_prompt path/to/dir -m` or `python -m files_to_prompt --help`.
12  - Package metadata is driven by `pyproject.toml`; bump versions there when preparing releases.
13  
14  ## Coding Style & Naming Conventions
15  - Follow standard Python 3.8+ conventions: 4-space indentation, snake_case for functions/variables, and meaningful, lowercase CLI option names.
16  - Keep functions small and reuse existing helpers (e.g., `process_path`, `print_as_markdown`) rather than adding branching in `cli()`.
17  - Prefer stdlib solutions; avoid new dependencies unless they deliver clear value.
18  - Match current import style (stdlib first, blank line, third-party).
19  
20  ## Testing Guidelines
21  - Primary test runner: `pytest`.
22  - Add or extend cases in `tests/test_files_to_prompt.py` when modifying CLI flags, output formats, or ignore rules.
23  - Include edge cases (hidden files, ignored paths, mixed encodings) and assert on concrete output strings, since the tool is output-centric.
24  - Run `pytest` before opening a PR; aim for coverage of new code paths rather than global percentage targets.
25  
26  ## Commit & Pull Request Guidelines
27  - Use concise, imperative commit subjects (e.g., `Add markdown line-number option`, `Refine ignore patterns`). One logical change per commit where possible.
28  - PRs should summarize intent, list main changes, and note testing performed (`pytest`). Link issues when they exist and include sample CLI output for user-facing changes.
29  - Keep diffs small and readable; prefer refactors in separate PRs from feature additions.
30  
31  ## Security & Configuration Tips
32  - The CLI reads files recursively; respect repository `.gitignore` handling and default hidden-file exclusion to avoid leaking secrets in prompts.
33  - When adding new options that affect file selection or output destinations, validate user inputs and maintain safe defaults (no overwriting unless requested).
```
./README.md
``````
  1  # files-to-prompt
  2  
  3  [![PyPI](https://img.shields.io/pypi/v/files-to-prompt.svg)](https://pypi.org/project/files-to-prompt/)
  4  [![Changelog](https://img.shields.io/github/v/release/simonw/files-to-prompt?include_prereleases&label=changelog)](https://github.com/simonw/files-to-prompt/releases)
  5  [![Tests](https://github.com/simonw/files-to-prompt/actions/workflows/test.yml/badge.svg)](https://github.com/simonw/files-to-prompt/actions/workflows/test.yml)
  6  [![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/simonw/files-to-prompt/blob/master/LICENSE)
  7  
  8  Concatenate a directory full of files into a single prompt for use with LLMs
  9  
 10  For background on this project see [Building files-to-prompt entirely using Claude 3 Opus](https://simonwillison.net/2024/Apr/8/files-to-prompt/).
 11  
 12  ## Installation
 13  
 14  Install this tool using `pip`:
 15  
 16  ```bash
 17  pip install files-to-prompt
 18  ```
 19  
 20  ## Usage
 21  
 22  To use `files-to-prompt`, provide the path to one or more files or directories you want to process:
 23  
 24  ```bash
 25  files-to-prompt path/to/file_or_directory [path/to/another/file_or_directory ...]
 26  ```
 27  
 28  This will output the contents of every file, with each file preceded by its relative path and separated by `---`.
 29  
 30  ### Options
 31  
 32  - `-e/--extension <extension>`: Only include files with the specified extension. Can be used multiple times.
 33  
 34    ```bash
 35    files-to-prompt path/to/directory -e txt -e md
 36    ```
 37  
 38  - `--include-hidden`: Include files and folders starting with `.` (hidden files and directories).
 39  
 40    ```bash
 41    files-to-prompt path/to/directory --include-hidden
 42    ```
 43  
 44  - `--ignore <pattern>`: Specify one or more patterns to ignore. Can be used multiple times. Patterns may match file names and directory names, unless you also specify `--ignore-files-only`. Pattern syntax uses [fnmatch](https://docs.python.org/3/library/fnmatch.html), which supports `*`, `?`, `[anychar]`, `[!notchars]` and `[?]` for special character literals.
 45    ```bash
 46    files-to-prompt path/to/directory --ignore "*.log" --ignore "temp*"
 47    ```
 48  
 49  - `--ignore-files-only`: Include directory paths which would otherwise be ignored by an `--ignore` pattern.
 50  
 51    ```bash
 52    files-to-prompt path/to/directory --ignore-files-only --ignore "*dir*"
 53    ```
 54  
 55  - `--ignore-gitignore`: Ignore `.gitignore` files and include all files.
 56  
 57    ```bash
 58    files-to-prompt path/to/directory --ignore-gitignore
 59    ```
 60  
 61  - `-c/--cxml`: Output in Claude XML format.
 62  
 63    ```bash
 64    files-to-prompt path/to/directory --cxml
 65    ```
 66  
 67  - `-m/--markdown`: Output as Markdown with fenced code blocks.
 68  
 69    ```bash
 70    files-to-prompt path/to/directory --markdown
 71    ```
 72  
 73  - `-o/--output <file>`: Write the output to a file instead of printing it to the console.
 74  
 75    ```bash
 76    files-to-prompt path/to/directory -o output.txt
 77    ```
 78  
 79  - `-n/--line-numbers`: Include line numbers in the output.
 80  
 81    ```bash
 82    files-to-prompt path/to/directory -n
 83    ```
 84    Example output:
 85    ```
 86    files_to_prompt/cli.py
 87    ---
 88      1  import os
 89      2  from fnmatch import fnmatch
 90      3
 91      4  import click
 92      ...
 93    ```
 94  
 95  - `-0/--null`: Use NUL character as separator when reading paths from stdin. Useful when filenames may contain spaces.
 96  
 97    ```bash
 98    find . -name "*.py" -print0 | files-to-prompt --null
 99    ```
100  
101  ### Example
102  
103  Suppose you have a directory structure like this:
104  
105  ```
106  my_directory/
107  ├── file1.txt
108  ├── file2.txt
109  ├── .hidden_file.txt
110  ├── temp.log
111  └── subdirectory/
112      └── file3.txt
113  ```
114  
115  Running `files-to-prompt my_directory` will output:
116  
117  ```
118  my_directory/file1.txt
119  ---
120  Contents of file1.txt
121  ---
122  my_directory/file2.txt
123  ---
124  Contents of file2.txt
125  ---
126  my_directory/subdirectory/file3.txt
127  ---
128  Contents of file3.txt
129  ---
130  ```
131  
132  If you run `files-to-prompt my_directory --include-hidden`, the output will also include `.hidden_file.txt`:
133  
134  ```
135  my_directory/.hidden_file.txt
136  ---
137  Contents of .hidden_file.txt
138  ---
139  ...
140  ```
141  
142  If you run `files-to-prompt my_directory --ignore "*.log"`, the output will exclude `temp.log`:
143  
144  ```
145  my_directory/file1.txt
146  ---
147  Contents of file1.txt
148  ---
149  my_directory/file2.txt
150  ---
151  Contents of file2.txt
152  ---
153  my_directory/subdirectory/file3.txt
154  ---
155  Contents of file3.txt
156  ---
157  ```
158  
159  If you run `files-to-prompt my_directory --ignore "sub*"`, the output will exclude all files in `subdirectory/` (unless you also specify `--ignore-files-only`):
160  
161  ```
162  my_directory/file1.txt
163  ---
164  Contents of file1.txt
165  ---
166  my_directory/file2.txt
167  ---
168  Contents of file2.txt
169  ---
170  ```
171  
172  ### Reading from stdin
173  
174  The tool can also read paths from standard input. This can be used to pipe in the output of another command:
175  
176  ```bash
177  # Find files modified in the last day
178  find . -mtime -1 | files-to-prompt
179  ```
180  
181  When using the `--null` (or `-0`) option, paths are expected to be NUL-separated (useful when dealing with filenames containing spaces):
182  
183  ```bash
184  find . -name "*.txt" -print0 | files-to-prompt --null
185  ```
186  
187  You can mix and match paths from command line arguments and stdin:
188  
189  ```bash
190  # Include files modified in the last day, and also include README.md
191  find . -mtime -1 | files-to-prompt README.md
192  ```
193  
194  ### Claude XML Output
195  
196  Anthropic has provided [specific guidelines](https://docs.anthropic.com/claude/docs/long-context-window-tips) for optimally structuring prompts to take advantage of Claude's extended context window.
197  
198  To structure the output in this way, use the optional `--cxml` flag, which will produce output like this:
199  
200  ```xml
201  <documents>
202  <document index="1">
203  <source>my_directory/file1.txt</source>
204  <document_content>
205  Contents of file1.txt
206  </document_content>
207  </document>
208  <document index="2">
209  <source>my_directory/file2.txt</source>
210  <document_content>
211  Contents of file2.txt
212  </document_content>
213  </document>
214  </documents>
215  ```
216  
217  ## --markdown fenced code block output
218  
219  The `--markdown` option will output the files as fenced code blocks, which can be useful for pasting into Markdown documents.
220  
221  ```bash
222  files-to-prompt path/to/directory --markdown
223  ```
224  The language tag will be guessed based on the filename.
225  
226  If the code itself contains triple backticks the wrapper around it will use one additional backtick.
227  
228  Example output:
229  `````
230  myfile.py
231  ```python
232  def my_function():
233      return "Hello, world!"
234  ```
235  other.js
236  ```javascript
237  function myFunction() {
238      return "Hello, world!";
239  }
240  ```
241  file_with_triple_backticks.md
242  ````markdown
243  This file has its own
244  ```
245  fenced code blocks
246  ```
247  Inside it.
248  ````
249  `````
250  
251  ## Development
252  
253  To contribute to this tool, first checkout the code. Then create a new virtual environment:
254  
255  ```bash
256  cd files-to-prompt
257  python -m venv venv
258  source venv/bin/activate
259  ```
260  
261  Now install the dependencies and test dependencies:
262  
263  ```bash
264  pip install -e '.[test]'
265  ```
266  
267  To run the tests:
268  
269  ```bash
270  pytest
271  ```
``````
./output.md
```

```
./pyproject.toml
```
 1  [project]
 2  name = "files-to-prompt"
 3  version = "0.6"
 4  description = "Concatenate a directory full of files into a single prompt for use with LLMs"
 5  readme = "README.md"
 6  authors = [{name = "Simon Willison"}]
 7  license = {text = "Apache-2.0"}
 8  requires-python = ">=3.8"
 9  classifiers = [
10      "License :: OSI Approved :: Apache Software License"
11  ]
12  dependencies = [
13      "click",
14      "pandoc>=2.4",
15      "pdfplumber",
16  ]
17  
18  [project.urls]
19  Homepage = "https://github.com/simonw/files-to-prompt"
20  Changelog = "https://github.com/simonw/files-to-prompt/releases"
21  Issues = "https://github.com/simonw/files-to-prompt/issues"
22  CI = "https://github.com/simonw/files-to-prompt/actions"
23  
24  [project.entry-points.console_scripts]
25  files-to-prompt = "files_to_prompt.cli:cli"
26  
27  [project.optional-dependencies]
28  test = ["pytest"]
```
./uv.lock
```
   1  version = 1
   2  revision = 3
   3  requires-python = ">=3.8"
   4  resolution-markers = [
   5      "python_full_version >= '3.10'",
   6      "python_full_version == '3.9.*'",
   7      "python_full_version < '3.9'",
   8  ]
   9  
  10  [[package]]
  11  name = "cffi"
  12  version = "1.17.1"
  13  source = { registry = "https://pypi.org/simple" }
  14  resolution-markers = [
  15      "python_full_version < '3.9'",
  16  ]
  17  dependencies = [
  18      { name = "pycparser", marker = "python_full_version < '3.9'" },
  19  ]
  20  sdist = { url = "https://files.pythonhosted.org/packages/fc/97/c783634659c2920c3fc70419e3af40972dbaf758daa229a7d6ea6135c90d/cffi-1.17.1.tar.gz", hash = "sha256:1c39c6016c32bc48dd54561950ebd6836e1670f2ae46128f67cf49e789c52824", size = 516621, upload-time = "2024-09-04T20:45:21.852Z" }
  21  wheels = [
  22      { url = "https://files.pythonhosted.org/packages/90/07/f44ca684db4e4f08a3fdc6eeb9a0d15dc6883efc7b8c90357fdbf74e186c/cffi-1.17.1-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:df8b1c11f177bc2313ec4b2d46baec87a5f3e71fc8b45dab2ee7cae86d9aba14", size = 182191, upload-time = "2024-09-04T20:43:30.027Z" },
  23      { url = "https://files.pythonhosted.org/packages/08/fd/cc2fedbd887223f9f5d170c96e57cbf655df9831a6546c1727ae13fa977a/cffi-1.17.1-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:8f2cdc858323644ab277e9bb925ad72ae0e67f69e804f4898c070998d50b1a67", size = 178592, upload-time = "2024-09-04T20:43:32.108Z" },
  24      { url = "https://files.pythonhosted.org/packages/de/cc/4635c320081c78d6ffc2cab0a76025b691a91204f4aa317d568ff9280a2d/cffi-1.17.1-cp310-cp310-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:edae79245293e15384b51f88b00613ba9f7198016a5948b5dddf4917d4d26382", size = 426024, upload-time = "2024-09-04T20:43:34.186Z" },
  25      { url = "https://files.pythonhosted.org/packages/b6/7b/3b2b250f3aab91abe5f8a51ada1b717935fdaec53f790ad4100fe2ec64d1/cffi-1.17.1-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:45398b671ac6d70e67da8e4224a065cec6a93541bb7aebe1b198a61b58c7b702", size = 448188, upload-time = "2024-09-04T20:43:36.286Z" },
  26      { url = "https://files.pythonhosted.org/packages/d3/48/1b9283ebbf0ec065148d8de05d647a986c5f22586b18120020452fff8f5d/cffi-1.17.1-cp310-cp310-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:ad9413ccdeda48c5afdae7e4fa2192157e991ff761e7ab8fdd8926f40b160cc3", size = 455571, upload-time = "2024-09-04T20:43:38.586Z" },
  27      { url = "https://files.pythonhosted.org/packages/40/87/3b8452525437b40f39ca7ff70276679772ee7e8b394934ff60e63b7b090c/cffi-1.17.1-cp310-cp310-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:5da5719280082ac6bd9aa7becb3938dc9f9cbd57fac7d2871717b1feb0902ab6", size = 436687, upload-time = "2024-09-04T20:43:40.084Z" },
  28      { url = "https://files.pythonhosted.org/packages/8d/fb/4da72871d177d63649ac449aec2e8a29efe0274035880c7af59101ca2232/cffi-1.17.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:2bb1a08b8008b281856e5971307cc386a8e9c5b625ac297e853d36da6efe9c17", size = 446211, upload-time = "2024-09-04T20:43:41.526Z" },
  29      { url = "https://files.pythonhosted.org/packages/ab/a0/62f00bcb411332106c02b663b26f3545a9ef136f80d5df746c05878f8c4b/cffi-1.17.1-cp310-cp310-musllinux_1_1_aarch64.whl", hash = "sha256:045d61c734659cc045141be4bae381a41d89b741f795af1dd018bfb532fd0df8", size = 461325, upload-time = "2024-09-04T20:43:43.117Z" },
  30      { url = "https://files.pythonhosted.org/packages/36/83/76127035ed2e7e27b0787604d99da630ac3123bfb02d8e80c633f218a11d/cffi-1.17.1-cp310-cp310-musllinux_1_1_i686.whl", hash = "sha256:6883e737d7d9e4899a8a695e00ec36bd4e5e4f18fabe0aca0efe0a4b44cdb13e", size = 438784, upload-time = "2024-09-04T20:43:45.256Z" },
  31      { url = "https://files.pythonhosted.org/packages/21/81/a6cd025db2f08ac88b901b745c163d884641909641f9b826e8cb87645942/cffi-1.17.1-cp310-cp310-musllinux_1_1_x86_64.whl", hash = "sha256:6b8b4a92e1c65048ff98cfe1f735ef8f1ceb72e3d5f0c25fdb12087a23da22be", size = 461564, upload-time = "2024-09-04T20:43:46.779Z" },
  32      { url = "https://files.pythonhosted.org/packages/f8/fe/4d41c2f200c4a457933dbd98d3cf4e911870877bd94d9656cc0fcb390681/cffi-1.17.1-cp310-cp310-win32.whl", hash = "sha256:c9c3d058ebabb74db66e431095118094d06abf53284d9c81f27300d0e0d8bc7c", size = 171804, upload-time = "2024-09-04T20:43:48.186Z" },
  33      { url = "https://files.pythonhosted.org/packages/d1/b6/0b0f5ab93b0df4acc49cae758c81fe4e5ef26c3ae2e10cc69249dfd8b3ab/cffi-1.17.1-cp310-cp310-win_amd64.whl", hash = "sha256:0f048dcf80db46f0098ccac01132761580d28e28bc0f78ae0d58048063317e15", size = 181299, upload-time = "2024-09-04T20:43:49.812Z" },
  34      { url = "https://files.pythonhosted.org/packages/6b/f4/927e3a8899e52a27fa57a48607ff7dc91a9ebe97399b357b85a0c7892e00/cffi-1.17.1-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:a45e3c6913c5b87b3ff120dcdc03f6131fa0065027d0ed7ee6190736a74cd401", size = 182264, upload-time = "2024-09-04T20:43:51.124Z" },
  35      { url = "https://files.pythonhosted.org/packages/6c/f5/6c3a8efe5f503175aaddcbea6ad0d2c96dad6f5abb205750d1b3df44ef29/cffi-1.17.1-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:30c5e0cb5ae493c04c8b42916e52ca38079f1b235c2f8ae5f4527b963c401caf", size = 178651, upload-time = "2024-09-04T20:43:52.872Z" },
  36      { url = "https://files.pythonhosted.org/packages/94/dd/a3f0118e688d1b1a57553da23b16bdade96d2f9bcda4d32e7d2838047ff7/cffi-1.17.1-cp311-cp311-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f75c7ab1f9e4aca5414ed4d8e5c0e303a34f4421f8a0d47a4d019ceff0ab6af4", size = 445259, upload-time = "2024-09-04T20:43:56.123Z" },
  37      { url = "https://files.pythonhosted.org/packages/2e/ea/70ce63780f096e16ce8588efe039d3c4f91deb1dc01e9c73a287939c79a6/cffi-1.17.1-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:a1ed2dd2972641495a3ec98445e09766f077aee98a1c896dcb4ad0d303628e41", size = 469200, upload-time = "2024-09-04T20:43:57.891Z" },
  38      { url = "https://files.pythonhosted.org/packages/1c/a0/a4fa9f4f781bda074c3ddd57a572b060fa0df7655d2a4247bbe277200146/cffi-1.17.1-cp311-cp311-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:46bf43160c1a35f7ec506d254e5c890f3c03648a4dbac12d624e4490a7046cd1", size = 477235, upload-time = "2024-09-04T20:44:00.18Z" },
  39      { url = "https://files.pythonhosted.org/packages/62/12/ce8710b5b8affbcdd5c6e367217c242524ad17a02fe5beec3ee339f69f85/cffi-1.17.1-cp311-cp311-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:a24ed04c8ffd54b0729c07cee15a81d964e6fee0e3d4d342a27b020d22959dc6", size = 459721, upload-time = "2024-09-04T20:44:01.585Z" },
  40      { url = "https://files.pythonhosted.org/packages/ff/6b/d45873c5e0242196f042d555526f92aa9e0c32355a1be1ff8c27f077fd37/cffi-1.17.1-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:610faea79c43e44c71e1ec53a554553fa22321b65fae24889706c0a84d4ad86d", size = 467242, upload-time = "2024-09-04T20:44:03.467Z" },
  41      { url = "https://files.pythonhosted.org/packages/1a/52/d9a0e523a572fbccf2955f5abe883cfa8bcc570d7faeee06336fbd50c9fc/cffi-1.17.1-cp311-cp311-musllinux_1_1_aarch64.whl", hash = "sha256:a9b15d491f3ad5d692e11f6b71f7857e7835eb677955c00cc0aefcd0669adaf6", size = 477999, upload-time = "2024-09-04T20:44:05.023Z" },
  42      { url = "https://files.pythonhosted.org/packages/44/74/f2a2460684a1a2d00ca799ad880d54652841a780c4c97b87754f660c7603/cffi-1.17.1-cp311-cp311-musllinux_1_1_i686.whl", hash = "sha256:de2ea4b5833625383e464549fec1bc395c1bdeeb5f25c4a3a82b5a8c756ec22f", size = 454242, upload-time = "2024-09-04T20:44:06.444Z" },
  43      { url = "https://files.pythonhosted.org/packages/f8/4a/34599cac7dfcd888ff54e801afe06a19c17787dfd94495ab0c8d35fe99fb/cffi-1.17.1-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:fc48c783f9c87e60831201f2cce7f3b2e4846bf4d8728eabe54d60700b318a0b", size = 478604, upload-time = "2024-09-04T20:44:08.206Z" },
  44      { url = "https://files.pythonhosted.org/packages/34/33/e1b8a1ba29025adbdcda5fb3a36f94c03d771c1b7b12f726ff7fef2ebe36/cffi-1.17.1-cp311-cp311-win32.whl", hash = "sha256:85a950a4ac9c359340d5963966e3e0a94a676bd6245a4b55bc43949eee26a655", size = 171727, upload-time = "2024-09-04T20:44:09.481Z" },
  45      { url = "https://files.pythonhosted.org/packages/3d/97/50228be003bb2802627d28ec0627837ac0bf35c90cf769812056f235b2d1/cffi-1.17.1-cp311-cp311-win_amd64.whl", hash = "sha256:caaf0640ef5f5517f49bc275eca1406b0ffa6aa184892812030f04c2abf589a0", size = 181400, upload-time = "2024-09-04T20:44:10.873Z" },
  46      { url = "https://files.pythonhosted.org/packages/5a/84/e94227139ee5fb4d600a7a4927f322e1d4aea6fdc50bd3fca8493caba23f/cffi-1.17.1-cp312-cp312-macosx_10_9_x86_64.whl", hash = "sha256:805b4371bf7197c329fcb3ead37e710d1bca9da5d583f5073b799d5c5bd1eee4", size = 183178, upload-time = "2024-09-04T20:44:12.232Z" },
  47      { url = "https://files.pythonhosted.org/packages/da/ee/fb72c2b48656111c4ef27f0f91da355e130a923473bf5ee75c5643d00cca/cffi-1.17.1-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:733e99bc2df47476e3848417c5a4540522f234dfd4ef3ab7fafdf555b082ec0c", size = 178840, upload-time = "2024-09-04T20:44:13.739Z" },
  48      { url = "https://files.pythonhosted.org/packages/cc/b6/db007700f67d151abadf508cbfd6a1884f57eab90b1bb985c4c8c02b0f28/cffi-1.17.1-cp312-cp312-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1257bdabf294dceb59f5e70c64a3e2f462c30c7ad68092d01bbbfb1c16b1ba36", size = 454803, upload-time = "2024-09-04T20:44:15.231Z" },
  49      { url = "https://files.pythonhosted.org/packages/1a/df/f8d151540d8c200eb1c6fba8cd0dfd40904f1b0682ea705c36e6c2e97ab3/cffi-1.17.1-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:da95af8214998d77a98cc14e3a3bd00aa191526343078b530ceb0bd710fb48a5", size = 478850, upload-time = "2024-09-04T20:44:17.188Z" },
  50      { url = "https://files.pythonhosted.org/packages/28/c0/b31116332a547fd2677ae5b78a2ef662dfc8023d67f41b2a83f7c2aa78b1/cffi-1.17.1-cp312-cp312-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:d63afe322132c194cf832bfec0dc69a99fb9bb6bbd550f161a49e9e855cc78ff", size = 485729, upload-time = "2024-09-04T20:44:18.688Z" },
  51      { url = "https://files.pythonhosted.org/packages/91/2b/9a1ddfa5c7f13cab007a2c9cc295b70fbbda7cb10a286aa6810338e60ea1/cffi-1.17.1-cp312-cp312-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:f79fc4fc25f1c8698ff97788206bb3c2598949bfe0fef03d299eb1b5356ada99", size = 471256, upload-time = "2024-09-04T20:44:20.248Z" },
  52      { url = "https://files.pythonhosted.org/packages/b2/d5/da47df7004cb17e4955df6a43d14b3b4ae77737dff8bf7f8f333196717bf/cffi-1.17.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:b62ce867176a75d03a665bad002af8e6d54644fad99a3c70905c543130e39d93", size = 479424, upload-time = "2024-09-04T20:44:21.673Z" },
  53      { url = "https://files.pythonhosted.org/packages/0b/ac/2a28bcf513e93a219c8a4e8e125534f4f6db03e3179ba1c45e949b76212c/cffi-1.17.1-cp312-cp312-musllinux_1_1_aarch64.whl", hash = "sha256:386c8bf53c502fff58903061338ce4f4950cbdcb23e2902d86c0f722b786bbe3", size = 484568, upload-time = "2024-09-04T20:44:23.245Z" },
  54      { url = "https://files.pythonhosted.org/packages/d4/38/ca8a4f639065f14ae0f1d9751e70447a261f1a30fa7547a828ae08142465/cffi-1.17.1-cp312-cp312-musllinux_1_1_x86_64.whl", hash = "sha256:4ceb10419a9adf4460ea14cfd6bc43d08701f0835e979bf821052f1805850fe8", size = 488736, upload-time = "2024-09-04T20:44:24.757Z" },
  55      { url = "https://files.pythonhosted.org/packages/86/c5/28b2d6f799ec0bdecf44dced2ec5ed43e0eb63097b0f58c293583b406582/cffi-1.17.1-cp312-cp312-win32.whl", hash = "sha256:a08d7e755f8ed21095a310a693525137cfe756ce62d066e53f502a83dc550f65", size = 172448, upload-time = "2024-09-04T20:44:26.208Z" },
  56      { url = "https://files.pythonhosted.org/packages/50/b9/db34c4755a7bd1cb2d1603ac3863f22bcecbd1ba29e5ee841a4bc510b294/cffi-1.17.1-cp312-cp312-win_amd64.whl", hash = "sha256:51392eae71afec0d0c8fb1a53b204dbb3bcabcb3c9b807eedf3e1e6ccf2de903", size = 181976, upload-time = "2024-09-04T20:44:27.578Z" },
  57      { url = "https://files.pythonhosted.org/packages/8d/f8/dd6c246b148639254dad4d6803eb6a54e8c85c6e11ec9df2cffa87571dbe/cffi-1.17.1-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:f3a2b4222ce6b60e2e8b337bb9596923045681d71e5a082783484d845390938e", size = 182989, upload-time = "2024-09-04T20:44:28.956Z" },
  58      { url = "https://files.pythonhosted.org/packages/8b/f1/672d303ddf17c24fc83afd712316fda78dc6fce1cd53011b839483e1ecc8/cffi-1.17.1-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:0984a4925a435b1da406122d4d7968dd861c1385afe3b45ba82b750f229811e2", size = 178802, upload-time = "2024-09-04T20:44:30.289Z" },
  59      { url = "https://files.pythonhosted.org/packages/0e/2d/eab2e858a91fdff70533cab61dcff4a1f55ec60425832ddfdc9cd36bc8af/cffi-1.17.1-cp313-cp313-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:d01b12eeeb4427d3110de311e1774046ad344f5b1a7403101878976ecd7a10f3", size = 454792, upload-time = "2024-09-04T20:44:32.01Z" },
  60      { url = "https://files.pythonhosted.org/packages/75/b2/fbaec7c4455c604e29388d55599b99ebcc250a60050610fadde58932b7ee/cffi-1.17.1-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:706510fe141c86a69c8ddc029c7910003a17353970cff3b904ff0686a5927683", size = 478893, upload-time = "2024-09-04T20:44:33.606Z" },
  61      { url = "https://files.pythonhosted.org/packages/4f/b7/6e4a2162178bf1935c336d4da8a9352cccab4d3a5d7914065490f08c0690/cffi-1.17.1-cp313-cp313-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:de55b766c7aa2e2a3092c51e0483d700341182f08e67c63630d5b6f200bb28e5", size = 485810, upload-time = "2024-09-04T20:44:35.191Z" },
  62      { url = "https://files.pythonhosted.org/packages/c7/8a/1d0e4a9c26e54746dc08c2c6c037889124d4f59dffd853a659fa545f1b40/cffi-1.17.1-cp313-cp313-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:c59d6e989d07460165cc5ad3c61f9fd8f1b4796eacbd81cee78957842b834af4", size = 471200, upload-time = "2024-09-04T20:44:36.743Z" },
  63      { url = "https://files.pythonhosted.org/packages/26/9f/1aab65a6c0db35f43c4d1b4f580e8df53914310afc10ae0397d29d697af4/cffi-1.17.1-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:dd398dbc6773384a17fe0d3e7eeb8d1a21c2200473ee6806bb5e6a8e62bb73dd", size = 479447, upload-time = "2024-09-04T20:44:38.492Z" },
  64      { url = "https://files.pythonhosted.org/packages/5f/e4/fb8b3dd8dc0e98edf1135ff067ae070bb32ef9d509d6cb0f538cd6f7483f/cffi-1.17.1-cp313-cp313-musllinux_1_1_aarch64.whl", hash = "sha256:3edc8d958eb099c634dace3c7e16560ae474aa3803a5df240542b305d14e14ed", size = 484358, upload-time = "2024-09-04T20:44:40.046Z" },
  65      { url = "https://files.pythonhosted.org/packages/f1/47/d7145bf2dc04684935d57d67dff9d6d795b2ba2796806bb109864be3a151/cffi-1.17.1-cp313-cp313-musllinux_1_1_x86_64.whl", hash = "sha256:72e72408cad3d5419375fc87d289076ee319835bdfa2caad331e377589aebba9", size = 488469, upload-time = "2024-09-04T20:44:41.616Z" },
  66      { url = "https://files.pythonhosted.org/packages/bf/ee/f94057fa6426481d663b88637a9a10e859e492c73d0384514a17d78ee205/cffi-1.17.1-cp313-cp313-win32.whl", hash = "sha256:e03eab0a8677fa80d646b5ddece1cbeaf556c313dcfac435ba11f107ba117b5d", size = 172475, upload-time = "2024-09-04T20:44:43.733Z" },
  67      { url = "https://files.pythonhosted.org/packages/7c/fc/6a8cb64e5f0324877d503c854da15d76c1e50eb722e320b15345c4d0c6de/cffi-1.17.1-cp313-cp313-win_amd64.whl", hash = "sha256:f6a16c31041f09ead72d69f583767292f750d24913dadacf5756b966aacb3f1a", size = 182009, upload-time = "2024-09-04T20:44:45.309Z" },
  68      { url = "https://files.pythonhosted.org/packages/48/08/15bf6b43ae9bd06f6b00ad8a91f5a8fe1069d4c9fab550a866755402724e/cffi-1.17.1-cp38-cp38-macosx_10_9_x86_64.whl", hash = "sha256:636062ea65bd0195bc012fea9321aca499c0504409f413dc88af450b57ffd03b", size = 182457, upload-time = "2024-09-04T20:44:47.892Z" },
  69      { url = "https://files.pythonhosted.org/packages/c2/5b/f1523dd545f92f7df468e5f653ffa4df30ac222f3c884e51e139878f1cb5/cffi-1.17.1-cp38-cp38-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:c7eac2ef9b63c79431bc4b25f1cd649d7f061a28808cbc6c47b534bd789ef964", size = 425932, upload-time = "2024-09-04T20:44:49.491Z" },
  70      { url = "https://files.pythonhosted.org/packages/53/93/7e547ab4105969cc8c93b38a667b82a835dd2cc78f3a7dad6130cfd41e1d/cffi-1.17.1-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:e221cf152cff04059d011ee126477f0d9588303eb57e88923578ace7baad17f9", size = 448585, upload-time = "2024-09-04T20:44:51.671Z" },
  71      { url = "https://files.pythonhosted.org/packages/56/c4/a308f2c332006206bb511de219efeff090e9d63529ba0a77aae72e82248b/cffi-1.17.1-cp38-cp38-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:31000ec67d4221a71bd3f67df918b1f88f676f1c3b535a7eb473255fdc0b83fc", size = 456268, upload-time = "2024-09-04T20:44:53.51Z" },
  72      { url = "https://files.pythonhosted.org/packages/ca/5b/b63681518265f2f4060d2b60755c1c77ec89e5e045fc3773b72735ddaad5/cffi-1.17.1-cp38-cp38-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:6f17be4345073b0a7b8ea599688f692ac3ef23ce28e5df79c04de519dbc4912c", size = 436592, upload-time = "2024-09-04T20:44:55.085Z" },
  73      { url = "https://files.pythonhosted.org/packages/bb/19/b51af9f4a4faa4a8ac5a0e5d5c2522dcd9703d07fac69da34a36c4d960d3/cffi-1.17.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:0e2b1fac190ae3ebfe37b979cc1ce69c81f4e4fe5746bb401dca63a9062cdaf1", size = 446512, upload-time = "2024-09-04T20:44:57.135Z" },
  74      { url = "https://files.pythonhosted.org/packages/e2/63/2bed8323890cb613bbecda807688a31ed11a7fe7afe31f8faaae0206a9a3/cffi-1.17.1-cp38-cp38-win32.whl", hash = "sha256:7596d6620d3fa590f677e9ee430df2958d2d6d6de2feeae5b20e82c00b76fbf8", size = 171576, upload-time = "2024-09-04T20:44:58.535Z" },
  75      { url = "https://files.pythonhosted.org/packages/2f/70/80c33b044ebc79527447fd4fbc5455d514c3bb840dede4455de97da39b4d/cffi-1.17.1-cp38-cp38-win_amd64.whl", hash = "sha256:78122be759c3f8a014ce010908ae03364d00a1f81ab5c7f4a7a5120607ea56e1", size = 181229, upload-time = "2024-09-04T20:44:59.963Z" },
  76      { url = "https://files.pythonhosted.org/packages/b9/ea/8bb50596b8ffbc49ddd7a1ad305035daa770202a6b782fc164647c2673ad/cffi-1.17.1-cp39-cp39-macosx_10_9_x86_64.whl", hash = "sha256:b2ab587605f4ba0bf81dc0cb08a41bd1c0a5906bd59243d56bad7668a6fc6c16", size = 182220, upload-time = "2024-09-04T20:45:01.577Z" },
  77      { url = "https://files.pythonhosted.org/packages/ae/11/e77c8cd24f58285a82c23af484cf5b124a376b32644e445960d1a4654c3a/cffi-1.17.1-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:28b16024becceed8c6dfbc75629e27788d8a3f9030691a1dbf9821a128b22c36", size = 178605, upload-time = "2024-09-04T20:45:03.837Z" },
  78      { url = "https://files.pythonhosted.org/packages/ed/65/25a8dc32c53bf5b7b6c2686b42ae2ad58743f7ff644844af7cdb29b49361/cffi-1.17.1-cp39-cp39-manylinux_2_12_i686.manylinux2010_i686.manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:1d599671f396c4723d016dbddb72fe8e0397082b0a77a4fab8028923bec050e8", size = 424910, upload-time = "2024-09-04T20:45:05.315Z" },
  79      { url = "https://files.pythonhosted.org/packages/42/7a/9d086fab7c66bd7c4d0f27c57a1b6b068ced810afc498cc8c49e0088661c/cffi-1.17.1-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:ca74b8dbe6e8e8263c0ffd60277de77dcee6c837a3d0881d8c1ead7268c9e576", size = 447200, upload-time = "2024-09-04T20:45:06.903Z" },
  80      { url = "https://files.pythonhosted.org/packages/da/63/1785ced118ce92a993b0ec9e0d0ac8dc3e5dbfbcaa81135be56c69cabbb6/cffi-1.17.1-cp39-cp39-manylinux_2_17_ppc64le.manylinux2014_ppc64le.whl", hash = "sha256:f7f5baafcc48261359e14bcd6d9bff6d4b28d9103847c9e136694cb0501aef87", size = 454565, upload-time = "2024-09-04T20:45:08.975Z" },
  81      { url = "https://files.pythonhosted.org/packages/74/06/90b8a44abf3556599cdec107f7290277ae8901a58f75e6fe8f970cd72418/cffi-1.17.1-cp39-cp39-manylinux_2_17_s390x.manylinux2014_s390x.whl", hash = "sha256:98e3969bcff97cae1b2def8ba499ea3d6f31ddfdb7635374834cf89a1a08ecf0", size = 435635, upload-time = "2024-09-04T20:45:10.64Z" },
  82      { url = "https://files.pythonhosted.org/packages/bd/62/a1f468e5708a70b1d86ead5bab5520861d9c7eacce4a885ded9faa7729c3/cffi-1.17.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:cdf5ce3acdfd1661132f2a9c19cac174758dc2352bfe37d98aa7512c6b7178b3", size = 445218, upload-time = "2024-09-04T20:45:12.366Z" },
  83      { url = "https://files.pythonhosted.org/packages/5b/95/b34462f3ccb09c2594aa782d90a90b045de4ff1f70148ee79c69d37a0a5a/cffi-1.17.1-cp39-cp39-musllinux_1_1_aarch64.whl", hash = "sha256:9755e4345d1ec879e3849e62222a18c7174d65a6a92d5b346b1863912168b595", size = 460486, upload-time = "2024-09-04T20:45:13.935Z" },
  84      { url = "https://files.pythonhosted.org/packages/fc/fc/a1e4bebd8d680febd29cf6c8a40067182b64f00c7d105f8f26b5bc54317b/cffi-1.17.1-cp39-cp39-musllinux_1_1_i686.whl", hash = "sha256:f1e22e8c4419538cb197e4dd60acc919d7696e5ef98ee4da4e01d3f8cfa4cc5a", size = 437911, upload-time = "2024-09-04T20:45:15.696Z" },
  85      { url = "https://files.pythonhosted.org/packages/e6/c3/21cab7a6154b6a5ea330ae80de386e7665254835b9e98ecc1340b3a7de9a/cffi-1.17.1-cp39-cp39-musllinux_1_1_x86_64.whl", hash = "sha256:c03e868a0b3bc35839ba98e74211ed2b05d2119be4e8a0f224fba9384f1fe02e", size = 460632, upload-time = "2024-09-04T20:45:17.284Z" },
  86      { url = "https://files.pythonhosted.org/packages/cb/b5/fd9f8b5a84010ca169ee49f4e4ad6f8c05f4e3545b72ee041dbbcb159882/cffi-1.17.1-cp39-cp39-win32.whl", hash = "sha256:e31ae45bc2e29f6b2abd0de1cc3b9d5205aa847cafaecb8af1476a609a2f6eb7", size = 171820, upload-time = "2024-09-04T20:45:18.762Z" },
  87      { url = "https://files.pythonhosted.org/packages/8c/52/b08750ce0bce45c143e1b5d7357ee8c55341b52bdef4b0f081af1eb248c2/cffi-1.17.1-cp39-cp39-win_amd64.whl", hash = "sha256:d016c76bdd850f3c626af19b0542c9677ba156e4ee4fccfdd7848803533ef662", size = 181290, upload-time = "2024-09-04T20:45:20.226Z" },
  88  ]
  89  
  90  [[package]]
  91  name = "cffi"
  92  version = "2.0.0"
  93  source = { registry = "https://pypi.org/simple" }
  94  resolution-markers = [
  95      "python_full_version >= '3.10'",
  96      "python_full_version == '3.9.*'",
  97  ]
  98  dependencies = [
  99      { name = "pycparser", marker = "python_full_version >= '3.9' and implementation_name != 'PyPy'" },
 100  ]
 101  sdist = { url = "https://files.pythonhosted.org/packages/eb/56/b1ba7935a17738ae8453301356628e8147c79dbb825bcbc73dc7401f9846/cffi-2.0.0.tar.gz", hash = "sha256:44d1b5909021139fe36001ae048dbdde8214afa20200eda0f64c068cac5d5529", size = 523588, upload-time = "2025-09-08T23:24:04.541Z" }
 102  wheels = [
 103      { url = "https://files.pythonhosted.org/packages/93/d7/516d984057745a6cd96575eea814fe1edd6646ee6efd552fb7b0921dec83/cffi-2.0.0-cp310-cp310-macosx_10_13_x86_64.whl", hash = "sha256:0cf2d91ecc3fcc0625c2c530fe004f82c110405f101548512cce44322fa8ac44", size = 184283, upload-time = "2025-09-08T23:22:08.01Z" },
 104      { url = "https://files.pythonhosted.org/packages/9e/84/ad6a0b408daa859246f57c03efd28e5dd1b33c21737c2db84cae8c237aa5/cffi-2.0.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:f73b96c41e3b2adedc34a7356e64c8eb96e03a3782b535e043a986276ce12a49", size = 180504, upload-time = "2025-09-08T23:22:10.637Z" },
 105      { url = "https://files.pythonhosted.org/packages/50/bd/b1a6362b80628111e6653c961f987faa55262b4002fcec42308cad1db680/cffi-2.0.0-cp310-cp310-manylinux1_i686.manylinux2014_i686.manylinux_2_17_i686.manylinux_2_5_i686.whl", hash = "sha256:53f77cbe57044e88bbd5ed26ac1d0514d2acf0591dd6bb02a3ae37f76811b80c", size = 208811, upload-time = "2025-09-08T23:22:12.267Z" },
 106      { url = "https://files.pythonhosted.org/packages/4f/27/6933a8b2562d7bd1fb595074cf99cc81fc3789f6a6c05cdabb46284a3188/cffi-2.0.0-cp310-cp310-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:3e837e369566884707ddaf85fc1744b47575005c0a229de3327f8f9a20f4efeb", size = 216402, upload-time = "2025-09-08T23:22:13.455Z" },
 107      { url = "https://files.pythonhosted.org/packages/05/eb/b86f2a2645b62adcfff53b0dd97e8dfafb5c8aa864bd0d9a2c2049a0d551/cffi-2.0.0-cp310-cp310-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:5eda85d6d1879e692d546a078b44251cdd08dd1cfb98dfb77b670c97cee49ea0", size = 203217, upload-time = "2025-09-08T23:22:14.596Z" },
 108      { url = "https://files.pythonhosted.org/packages/9f/e0/6cbe77a53acf5acc7c08cc186c9928864bd7c005f9efd0d126884858a5fe/cffi-2.0.0-cp310-cp310-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:9332088d75dc3241c702d852d4671613136d90fa6881da7d770a483fd05248b4", size = 203079, upload-time = "2025-09-08T23:22:15.769Z" },
 109      { url = "https://files.pythonhosted.org/packages/98/29/9b366e70e243eb3d14a5cb488dfd3a0b6b2f1fb001a203f653b93ccfac88/cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:fc7de24befaeae77ba923797c7c87834c73648a05a4bde34b3b7e5588973a453", size = 216475, upload-time = "2025-09-08T23:22:17.427Z" },
 110      { url = "https://files.pythonhosted.org/packages/21/7a/13b24e70d2f90a322f2900c5d8e1f14fa7e2a6b3332b7309ba7b2ba51a5a/cffi-2.0.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:cf364028c016c03078a23b503f02058f1814320a56ad535686f90565636a9495", size = 218829, upload-time = "2025-09-08T23:22:19.069Z" },
 111      { url = "https://files.pythonhosted.org/packages/60/99/c9dc110974c59cc981b1f5b66e1d8af8af764e00f0293266824d9c4254bc/cffi-2.0.0-cp310-cp310-musllinux_1_2_i686.whl", hash = "sha256:e11e82b744887154b182fd3e7e8512418446501191994dbf9c9fc1f32cc8efd5", size = 211211, upload-time = "2025-09-08T23:22:20.588Z" },
 112      { url = "https://files.pythonhosted.org/packages/49/72/ff2d12dbf21aca1b32a40ed792ee6b40f6dc3a9cf1644bd7ef6e95e0ac5e/cffi-2.0.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:8ea985900c5c95ce9db1745f7933eeef5d314f0565b27625d9a10ec9881e1bfb", size = 218036, upload-time = "2025-09-08T23:22:22.143Z" },
 113      { url = "https://files.pythonhosted.org/packages/e2/cc/027d7fb82e58c48ea717149b03bcadcbdc293553edb283af792bd4bcbb3f/cffi-2.0.0-cp310-cp310-win32.whl", hash = "sha256:1f72fb8906754ac8a2cc3f9f5aaa298070652a0ffae577e0ea9bd480dc3c931a", size = 172184, upload-time = "2025-09-08T23:22:23.328Z" },
 114      { url = "https://files.pythonhosted.org/packages/33/fa/072dd15ae27fbb4e06b437eb6e944e75b068deb09e2a2826039e49ee2045/cffi-2.0.0-cp310-cp310-win_amd64.whl", hash = "sha256:b18a3ed7d5b3bd8d9ef7a8cb226502c6bf8308df1525e1cc676c3680e7176739", size = 182790, upload-time = "2025-09-08T23:22:24.752Z" },
 115      { url = "https://files.pythonhosted.org/packages/12/4a/3dfd5f7850cbf0d06dc84ba9aa00db766b52ca38d8b86e3a38314d52498c/cffi-2.0.0-cp311-cp311-macosx_10_13_x86_64.whl", hash = "sha256:b4c854ef3adc177950a8dfc81a86f5115d2abd545751a304c5bcf2c2c7283cfe", size = 184344, upload-time = "2025-09-08T23:22:26.456Z" },
 116      { url = "https://files.pythonhosted.org/packages/4f/8b/f0e4c441227ba756aafbe78f117485b25bb26b1c059d01f137fa6d14896b/cffi-2.0.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:2de9a304e27f7596cd03d16f1b7c72219bd944e99cc52b84d0145aefb07cbd3c", size = 180560, upload-time = "2025-09-08T23:22:28.197Z" },
 117      { url = "https://files.pythonhosted.org/packages/b1/b7/1200d354378ef52ec227395d95c2576330fd22a869f7a70e88e1447eb234/cffi-2.0.0-cp311-cp311-manylinux1_i686.manylinux2014_i686.manylinux_2_17_i686.manylinux_2_5_i686.whl", hash = "sha256:baf5215e0ab74c16e2dd324e8ec067ef59e41125d3eade2b863d294fd5035c92", size = 209613, upload-time = "2025-09-08T23:22:29.475Z" },
 118      { url = "https://files.pythonhosted.org/packages/b8/56/6033f5e86e8cc9bb629f0077ba71679508bdf54a9a5e112a3c0b91870332/cffi-2.0.0-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:730cacb21e1bdff3ce90babf007d0a0917cc3e6492f336c2f0134101e0944f93", size = 216476, upload-time = "2025-09-08T23:22:31.063Z" },
 119      { url = "https://files.pythonhosted.org/packages/dc/7f/55fecd70f7ece178db2f26128ec41430d8720f2d12ca97bf8f0a628207d5/cffi-2.0.0-cp311-cp311-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:6824f87845e3396029f3820c206e459ccc91760e8fa24422f8b0c3d1731cbec5", size = 203374, upload-time = "2025-09-08T23:22:32.507Z" },
 120      { url = "https://files.pythonhosted.org/packages/84/ef/a7b77c8bdc0f77adc3b46888f1ad54be8f3b7821697a7b89126e829e676a/cffi-2.0.0-cp311-cp311-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:9de40a7b0323d889cf8d23d1ef214f565ab154443c42737dfe52ff82cf857664", size = 202597, upload-time = "2025-09-08T23:22:34.132Z" },
 121      { url = "https://files.pythonhosted.org/packages/d7/91/500d892b2bf36529a75b77958edfcd5ad8e2ce4064ce2ecfeab2125d72d1/cffi-2.0.0-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:8941aaadaf67246224cee8c3803777eed332a19d909b47e29c9842ef1e79ac26", size = 215574, upload-time = "2025-09-08T23:22:35.443Z" },
 122      { url = "https://files.pythonhosted.org/packages/44/64/58f6255b62b101093d5df22dcb752596066c7e89dd725e0afaed242a61be/cffi-2.0.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:a05d0c237b3349096d3981b727493e22147f934b20f6f125a3eba8f994bec4a9", size = 218971, upload-time = "2025-09-08T23:22:36.805Z" },
 123      { url = "https://files.pythonhosted.org/packages/ab/49/fa72cebe2fd8a55fbe14956f9970fe8eb1ac59e5df042f603ef7c8ba0adc/cffi-2.0.0-cp311-cp311-musllinux_1_2_i686.whl", hash = "sha256:94698a9c5f91f9d138526b48fe26a199609544591f859c870d477351dc7b2414", size = 211972, upload-time = "2025-09-08T23:22:38.436Z" },
 124      { url = "https://files.pythonhosted.org/packages/0b/28/dd0967a76aab36731b6ebfe64dec4e981aff7e0608f60c2d46b46982607d/cffi-2.0.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:5fed36fccc0612a53f1d4d9a816b50a36702c28a2aa880cb8a122b3466638743", size = 217078, upload-time = "2025-09-08T23:22:39.776Z" },
 125      { url = "https://files.pythonhosted.org/packages/2b/c0/015b25184413d7ab0a410775fdb4a50fca20f5589b5dab1dbbfa3baad8ce/cffi-2.0.0-cp311-cp311-win32.whl", hash = "sha256:c649e3a33450ec82378822b3dad03cc228b8f5963c0c12fc3b1e0ab940f768a5", size = 172076, upload-time = "2025-09-08T23:22:40.95Z" },
 126      { url = "https://files.pythonhosted.org/packages/ae/8f/dc5531155e7070361eb1b7e4c1a9d896d0cb21c49f807a6c03fd63fc877e/cffi-2.0.0-cp311-cp311-win_amd64.whl", hash = "sha256:66f011380d0e49ed280c789fbd08ff0d40968ee7b665575489afa95c98196ab5", size = 182820, upload-time = "2025-09-08T23:22:42.463Z" },
 127      { url = "https://files.pythonhosted.org/packages/95/5c/1b493356429f9aecfd56bc171285a4c4ac8697f76e9bbbbb105e537853a1/cffi-2.0.0-cp311-cp311-win_arm64.whl", hash = "sha256:c6638687455baf640e37344fe26d37c404db8b80d037c3d29f58fe8d1c3b194d", size = 177635, upload-time = "2025-09-08T23:22:43.623Z" },
 128      { url = "https://files.pythonhosted.org/packages/ea/47/4f61023ea636104d4f16ab488e268b93008c3d0bb76893b1b31db1f96802/cffi-2.0.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:6d02d6655b0e54f54c4ef0b94eb6be0607b70853c45ce98bd278dc7de718be5d", size = 185271, upload-time = "2025-09-08T23:22:44.795Z" },
 129      { url = "https://files.pythonhosted.org/packages/df/a2/781b623f57358e360d62cdd7a8c681f074a71d445418a776eef0aadb4ab4/cffi-2.0.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:8eca2a813c1cb7ad4fb74d368c2ffbbb4789d377ee5bb8df98373c2cc0dee76c", size = 181048, upload-time = "2025-09-08T23:22:45.938Z" },
 130      { url = "https://files.pythonhosted.org/packages/ff/df/a4f0fbd47331ceeba3d37c2e51e9dfc9722498becbeec2bd8bc856c9538a/cffi-2.0.0-cp312-cp312-manylinux1_i686.manylinux2014_i686.manylinux_2_17_i686.manylinux_2_5_i686.whl", hash = "sha256:21d1152871b019407d8ac3985f6775c079416c282e431a4da6afe7aefd2bccbe", size = 212529, upload-time = "2025-09-08T23:22:47.349Z" },
 131      { url = "https://files.pythonhosted.org/packages/d5/72/12b5f8d3865bf0f87cf1404d8c374e7487dcf097a1c91c436e72e6badd83/cffi-2.0.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:b21e08af67b8a103c71a250401c78d5e0893beff75e28c53c98f4de42f774062", size = 220097, upload-time = "2025-09-08T23:22:48.677Z" },
 132      { url = "https://files.pythonhosted.org/packages/c2/95/7a135d52a50dfa7c882ab0ac17e8dc11cec9d55d2c18dda414c051c5e69e/cffi-2.0.0-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:1e3a615586f05fc4065a8b22b8152f0c1b00cdbc60596d187c2a74f9e3036e4e", size = 207983, upload-time = "2025-09-08T23:22:50.06Z" },
 133      { url = "https://files.pythonhosted.org/packages/3a/c8/15cb9ada8895957ea171c62dc78ff3e99159ee7adb13c0123c001a2546c1/cffi-2.0.0-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:81afed14892743bbe14dacb9e36d9e0e504cd204e0b165062c488942b9718037", size = 206519, upload-time = "2025-09-08T23:22:51.364Z" },
 134      { url = "https://files.pythonhosted.org/packages/78/2d/7fa73dfa841b5ac06c7b8855cfc18622132e365f5b81d02230333ff26e9e/cffi-2.0.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:3e17ed538242334bf70832644a32a7aae3d83b57567f9fd60a26257e992b79ba", size = 219572, upload-time = "2025-09-08T23:22:52.902Z" },
 135      { url = "https://files.pythonhosted.org/packages/07/e0/267e57e387b4ca276b90f0434ff88b2c2241ad72b16d31836adddfd6031b/cffi-2.0.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:3925dd22fa2b7699ed2617149842d2e6adde22b262fcbfada50e3d195e4b3a94", size = 222963, upload-time = "2025-09-08T23:22:54.518Z" },
 136      { url = "https://files.pythonhosted.org/packages/b6/75/1f2747525e06f53efbd878f4d03bac5b859cbc11c633d0fb81432d98a795/cffi-2.0.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:2c8f814d84194c9ea681642fd164267891702542f028a15fc97d4674b6206187", size = 221361, upload-time = "2025-09-08T23:22:55.867Z" },
 137      { url = "https://files.pythonhosted.org/packages/7b/2b/2b6435f76bfeb6bbf055596976da087377ede68df465419d192acf00c437/cffi-2.0.0-cp312-cp312-win32.whl", hash = "sha256:da902562c3e9c550df360bfa53c035b2f241fed6d9aef119048073680ace4a18", size = 172932, upload-time = "2025-09-08T23:22:57.188Z" },
 138      { url = "https://files.pythonhosted.org/packages/f8/ed/13bd4418627013bec4ed6e54283b1959cf6db888048c7cf4b4c3b5b36002/cffi-2.0.0-cp312-cp312-win_amd64.whl", hash = "sha256:da68248800ad6320861f129cd9c1bf96ca849a2771a59e0344e88681905916f5", size = 183557, upload-time = "2025-09-08T23:22:58.351Z" },
 139      { url = "https://files.pythonhosted.org/packages/95/31/9f7f93ad2f8eff1dbc1c3656d7ca5bfd8fb52c9d786b4dcf19b2d02217fa/cffi-2.0.0-cp312-cp312-win_arm64.whl", hash = "sha256:4671d9dd5ec934cb9a73e7ee9676f9362aba54f7f34910956b84d727b0d73fb6", size = 177762, upload-time = "2025-09-08T23:22:59.668Z" },
 140      { url = "https://files.pythonhosted.org/packages/4b/8d/a0a47a0c9e413a658623d014e91e74a50cdd2c423f7ccfd44086ef767f90/cffi-2.0.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:00bdf7acc5f795150faa6957054fbbca2439db2f775ce831222b66f192f03beb", size = 185230, upload-time = "2025-09-08T23:23:00.879Z" },
 141      { url = "https://files.pythonhosted.org/packages/4a/d2/a6c0296814556c68ee32009d9c2ad4f85f2707cdecfd7727951ec228005d/cffi-2.0.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:45d5e886156860dc35862657e1494b9bae8dfa63bf56796f2fb56e1679fc0bca", size = 181043, upload-time = "2025-09-08T23:23:02.231Z" },
 142      { url = "https://files.pythonhosted.org/packages/b0/1e/d22cc63332bd59b06481ceaac49d6c507598642e2230f201649058a7e704/cffi-2.0.0-cp313-cp313-manylinux1_i686.manylinux2014_i686.manylinux_2_17_i686.manylinux_2_5_i686.whl", hash = "sha256:07b271772c100085dd28b74fa0cd81c8fb1a3ba18b21e03d7c27f3436a10606b", size = 212446, upload-time = "2025-09-08T23:23:03.472Z" },
 143      { url = "https://files.pythonhosted.org/packages/a9/f5/a2c23eb03b61a0b8747f211eb716446c826ad66818ddc7810cc2cc19b3f2/cffi-2.0.0-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:d48a880098c96020b02d5a1f7d9251308510ce8858940e6fa99ece33f610838b", size = 220101, upload-time = "2025-09-08T23:23:04.792Z" },
 144      { url = "https://files.pythonhosted.org/packages/f2/7f/e6647792fc5850d634695bc0e6ab4111ae88e89981d35ac269956605feba/cffi-2.0.0-cp313-cp313-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:f93fd8e5c8c0a4aa1f424d6173f14a892044054871c771f8566e4008eaa359d2", size = 207948, upload-time = "2025-09-08T23:23:06.127Z" },
 145      { url = "https://files.pythonhosted.org/packages/cb/1e/a5a1bd6f1fb30f22573f76533de12a00bf274abcdc55c8edab639078abb6/cffi-2.0.0-cp313-cp313-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:dd4f05f54a52fb558f1ba9f528228066954fee3ebe629fc1660d874d040ae5a3", size = 206422, upload-time = "2025-09-08T23:23:07.753Z" },
 146      { url = "https://files.pythonhosted.org/packages/98/df/0a1755e750013a2081e863e7cd37e0cdd02664372c754e5560099eb7aa44/cffi-2.0.0-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:c8d3b5532fc71b7a77c09192b4a5a200ea992702734a2e9279a37f2478236f26", size = 219499, upload-time = "2025-09-08T23:23:09.648Z" },
 147      { url = "https://files.pythonhosted.org/packages/50/e1/a969e687fcf9ea58e6e2a928ad5e2dd88cc12f6f0ab477e9971f2309b57c/cffi-2.0.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:d9b29c1f0ae438d5ee9acb31cadee00a58c46cc9c0b2f9038c6b0b3470877a8c", size = 222928, upload-time = "2025-09-08T23:23:10.928Z" },
 148      { url = "https://files.pythonhosted.org/packages/36/54/0362578dd2c9e557a28ac77698ed67323ed5b9775ca9d3fe73fe191bb5d8/cffi-2.0.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:6d50360be4546678fc1b79ffe7a66265e28667840010348dd69a314145807a1b", size = 221302, upload-time = "2025-09-08T23:23:12.42Z" },
 149      { url = "https://files.pythonhosted.org/packages/eb/6d/bf9bda840d5f1dfdbf0feca87fbdb64a918a69bca42cfa0ba7b137c48cb8/cffi-2.0.0-cp313-cp313-win32.whl", hash = "sha256:74a03b9698e198d47562765773b4a8309919089150a0bb17d829ad7b44b60d27", size = 172909, upload-time = "2025-09-08T23:23:14.32Z" },
 150      { url = "https://files.pythonhosted.org/packages/37/18/6519e1ee6f5a1e579e04b9ddb6f1676c17368a7aba48299c3759bbc3c8b3/cffi-2.0.0-cp313-cp313-win_amd64.whl", hash = "sha256:19f705ada2530c1167abacb171925dd886168931e0a7b78f5bffcae5c6b5be75", size = 183402, upload-time = "2025-09-08T23:23:15.535Z" },
 151      { url = "https://files.pythonhosted.org/packages/cb/0e/02ceeec9a7d6ee63bb596121c2c8e9b3a9e150936f4fbef6ca1943e6137c/cffi-2.0.0-cp313-cp313-win_arm64.whl", hash = "sha256:256f80b80ca3853f90c21b23ee78cd008713787b1b1e93eae9f3d6a7134abd91", size = 177780, upload-time = "2025-09-08T23:23:16.761Z" },
 152      { url = "https://files.pythonhosted.org/packages/92/c4/3ce07396253a83250ee98564f8d7e9789fab8e58858f35d07a9a2c78de9f/cffi-2.0.0-cp314-cp314-macosx_10_13_x86_64.whl", hash = "sha256:fc33c5141b55ed366cfaad382df24fe7dcbc686de5be719b207bb248e3053dc5", size = 185320, upload-time = "2025-09-08T23:23:18.087Z" },
 153      { url = "https://files.pythonhosted.org/packages/59/dd/27e9fa567a23931c838c6b02d0764611c62290062a6d4e8ff7863daf9730/cffi-2.0.0-cp314-cp314-macosx_11_0_arm64.whl", hash = "sha256:c654de545946e0db659b3400168c9ad31b5d29593291482c43e3564effbcee13", size = 181487, upload-time = "2025-09-08T23:23:19.622Z" },
 154      { url = "https://files.pythonhosted.org/packages/d6/43/0e822876f87ea8a4ef95442c3d766a06a51fc5298823f884ef87aaad168c/cffi-2.0.0-cp314-cp314-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:24b6f81f1983e6df8db3adc38562c83f7d4a0c36162885ec7f7b77c7dcbec97b", size = 220049, upload-time = "2025-09-08T23:23:20.853Z" },
 155      { url = "https://files.pythonhosted.org/packages/b4/89/76799151d9c2d2d1ead63c2429da9ea9d7aac304603de0c6e8764e6e8e70/cffi-2.0.0-cp314-cp314-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:12873ca6cb9b0f0d3a0da705d6086fe911591737a59f28b7936bdfed27c0d47c", size = 207793, upload-time = "2025-09-08T23:23:22.08Z" },
 156      { url = "https://files.pythonhosted.org/packages/bb/dd/3465b14bb9e24ee24cb88c9e3730f6de63111fffe513492bf8c808a3547e/cffi-2.0.0-cp314-cp314-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:d9b97165e8aed9272a6bb17c01e3cc5871a594a446ebedc996e2397a1c1ea8ef", size = 206300, upload-time = "2025-09-08T23:23:23.314Z" },
 157      { url = "https://files.pythonhosted.org/packages/47/d9/d83e293854571c877a92da46fdec39158f8d7e68da75bf73581225d28e90/cffi-2.0.0-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:afb8db5439b81cf9c9d0c80404b60c3cc9c3add93e114dcae767f1477cb53775", size = 219244, upload-time = "2025-09-08T23:23:24.541Z" },
 158      { url = "https://files.pythonhosted.org/packages/2b/0f/1f177e3683aead2bb00f7679a16451d302c436b5cbf2505f0ea8146ef59e/cffi-2.0.0-cp314-cp314-musllinux_1_2_aarch64.whl", hash = "sha256:737fe7d37e1a1bffe70bd5754ea763a62a066dc5913ca57e957824b72a85e205", size = 222828, upload-time = "2025-09-08T23:23:26.143Z" },
 159      { url = "https://files.pythonhosted.org/packages/c6/0f/cafacebd4b040e3119dcb32fed8bdef8dfe94da653155f9d0b9dc660166e/cffi-2.0.0-cp314-cp314-musllinux_1_2_x86_64.whl", hash = "sha256:38100abb9d1b1435bc4cc340bb4489635dc2f0da7456590877030c9b3d40b0c1", size = 220926, upload-time = "2025-09-08T23:23:27.873Z" },
 160      { url = "https://files.pythonhosted.org/packages/3e/aa/df335faa45b395396fcbc03de2dfcab242cd61a9900e914fe682a59170b1/cffi-2.0.0-cp314-cp314-win32.whl", hash = "sha256:087067fa8953339c723661eda6b54bc98c5625757ea62e95eb4898ad5e776e9f", size = 175328, upload-time = "2025-09-08T23:23:44.61Z" },
 161      { url = "https://files.pythonhosted.org/packages/bb/92/882c2d30831744296ce713f0feb4c1cd30f346ef747b530b5318715cc367/cffi-2.0.0-cp314-cp314-win_amd64.whl", hash = "sha256:203a48d1fb583fc7d78a4c6655692963b860a417c0528492a6bc21f1aaefab25", size = 185650, upload-time = "2025-09-08T23:23:45.848Z" },
 162      { url = "https://files.pythonhosted.org/packages/9f/2c/98ece204b9d35a7366b5b2c6539c350313ca13932143e79dc133ba757104/cffi-2.0.0-cp314-cp314-win_arm64.whl", hash = "sha256:dbd5c7a25a7cb98f5ca55d258b103a2054f859a46ae11aaf23134f9cc0d356ad", size = 180687, upload-time = "2025-09-08T23:23:47.105Z" },
 163      { url = "https://files.pythonhosted.org/packages/3e/61/c768e4d548bfa607abcda77423448df8c471f25dbe64fb2ef6d555eae006/cffi-2.0.0-cp314-cp314t-macosx_10_13_x86_64.whl", hash = "sha256:9a67fc9e8eb39039280526379fb3a70023d77caec1852002b4da7e8b270c4dd9", size = 188773, upload-time = "2025-09-08T23:23:29.347Z" },
 164      { url = "https://files.pythonhosted.org/packages/2c/ea/5f76bce7cf6fcd0ab1a1058b5af899bfbef198bea4d5686da88471ea0336/cffi-2.0.0-cp314-cp314t-macosx_11_0_arm64.whl", hash = "sha256:7a66c7204d8869299919db4d5069a82f1561581af12b11b3c9f48c584eb8743d", size = 185013, upload-time = "2025-09-08T23:23:30.63Z" },
 165      { url = "https://files.pythonhosted.org/packages/be/b4/c56878d0d1755cf9caa54ba71e5d049479c52f9e4afc230f06822162ab2f/cffi-2.0.0-cp314-cp314t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:7cc09976e8b56f8cebd752f7113ad07752461f48a58cbba644139015ac24954c", size = 221593, upload-time = "2025-09-08T23:23:31.91Z" },
 166      { url = "https://files.pythonhosted.org/packages/e0/0d/eb704606dfe8033e7128df5e90fee946bbcb64a04fcdaa97321309004000/cffi-2.0.0-cp314-cp314t-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:92b68146a71df78564e4ef48af17551a5ddd142e5190cdf2c5624d0c3ff5b2e8", size = 209354, upload-time = "2025-09-08T23:23:33.214Z" },
 167      { url = "https://files.pythonhosted.org/packages/d8/19/3c435d727b368ca475fb8742ab97c9cb13a0de600ce86f62eab7fa3eea60/cffi-2.0.0-cp314-cp314t-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:b1e74d11748e7e98e2f426ab176d4ed720a64412b6a15054378afdb71e0f37dc", size = 208480, upload-time = "2025-09-08T23:23:34.495Z" },
 168      { url = "https://files.pythonhosted.org/packages/d0/44/681604464ed9541673e486521497406fadcc15b5217c3e326b061696899a/cffi-2.0.0-cp314-cp314t-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:28a3a209b96630bca57cce802da70c266eb08c6e97e5afd61a75611ee6c64592", size = 221584, upload-time = "2025-09-08T23:23:36.096Z" },
 169      { url = "https://files.pythonhosted.org/packages/25/8e/342a504ff018a2825d395d44d63a767dd8ebc927ebda557fecdaca3ac33a/cffi-2.0.0-cp314-cp314t-musllinux_1_2_aarch64.whl", hash = "sha256:7553fb2090d71822f02c629afe6042c299edf91ba1bf94951165613553984512", size = 224443, upload-time = "2025-09-08T23:23:37.328Z" },
 170      { url = "https://files.pythonhosted.org/packages/e1/5e/b666bacbbc60fbf415ba9988324a132c9a7a0448a9a8f125074671c0f2c3/cffi-2.0.0-cp314-cp314t-musllinux_1_2_x86_64.whl", hash = "sha256:6c6c373cfc5c83a975506110d17457138c8c63016b563cc9ed6e056a82f13ce4", size = 223437, upload-time = "2025-09-08T23:23:38.945Z" },
 171      { url = "https://files.pythonhosted.org/packages/a0/1d/ec1a60bd1a10daa292d3cd6bb0b359a81607154fb8165f3ec95fe003b85c/cffi-2.0.0-cp314-cp314t-win32.whl", hash = "sha256:1fc9ea04857caf665289b7a75923f2c6ed559b8298a1b8c49e59f7dd95c8481e", size = 180487, upload-time = "2025-09-08T23:23:40.423Z" },
 172      { url = "https://files.pythonhosted.org/packages/bf/41/4c1168c74fac325c0c8156f04b6749c8b6a8f405bbf91413ba088359f60d/cffi-2.0.0-cp314-cp314t-win_amd64.whl", hash = "sha256:d68b6cef7827e8641e8ef16f4494edda8b36104d79773a334beaa1e3521430f6", size = 191726, upload-time = "2025-09-08T23:23:41.742Z" },
 173      { url = "https://files.pythonhosted.org/packages/ae/3a/dbeec9d1ee0844c679f6bb5d6ad4e9f198b1224f4e7a32825f47f6192b0c/cffi-2.0.0-cp314-cp314t-win_arm64.whl", hash = "sha256:0a1527a803f0a659de1af2e1fd700213caba79377e27e4693648c2923da066f9", size = 184195, upload-time = "2025-09-08T23:23:43.004Z" },
 174      { url = "https://files.pythonhosted.org/packages/c0/cc/08ed5a43f2996a16b462f64a7055c6e962803534924b9b2f1371d8c00b7b/cffi-2.0.0-cp39-cp39-macosx_10_13_x86_64.whl", hash = "sha256:fe562eb1a64e67dd297ccc4f5addea2501664954f2692b69a76449ec7913ecbf", size = 184288, upload-time = "2025-09-08T23:23:48.404Z" },
 175      { url = "https://files.pythonhosted.org/packages/3d/de/38d9726324e127f727b4ecc376bc85e505bfe61ef130eaf3f290c6847dd4/cffi-2.0.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:de8dad4425a6ca6e4e5e297b27b5c824ecc7581910bf9aee86cb6835e6812aa7", size = 180509, upload-time = "2025-09-08T23:23:49.73Z" },
 176      { url = "https://files.pythonhosted.org/packages/9b/13/c92e36358fbcc39cf0962e83223c9522154ee8630e1df7c0b3a39a8124e2/cffi-2.0.0-cp39-cp39-manylinux1_i686.manylinux2014_i686.manylinux_2_17_i686.manylinux_2_5_i686.whl", hash = "sha256:4647afc2f90d1ddd33441e5b0e85b16b12ddec4fca55f0d9671fef036ecca27c", size = 208813, upload-time = "2025-09-08T23:23:51.263Z" },
 177      { url = "https://files.pythonhosted.org/packages/15/12/a7a79bd0df4c3bff744b2d7e52cc1b68d5e7e427b384252c42366dc1ecbc/cffi-2.0.0-cp39-cp39-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:3f4d46d8b35698056ec29bca21546e1551a205058ae1a181d871e278b0b28165", size = 216498, upload-time = "2025-09-08T23:23:52.494Z" },
 178      { url = "https://files.pythonhosted.org/packages/a3/ad/5c51c1c7600bdd7ed9a24a203ec255dccdd0ebf4527f7b922a0bde2fb6ed/cffi-2.0.0-cp39-cp39-manylinux2014_ppc64le.manylinux_2_17_ppc64le.whl", hash = "sha256:e6e73b9e02893c764e7e8d5bb5ce277f1a009cd5243f8228f75f842bf937c534", size = 203243, upload-time = "2025-09-08T23:23:53.836Z" },
 179      { url = "https://files.pythonhosted.org/packages/32/f2/81b63e288295928739d715d00952c8c6034cb6c6a516b17d37e0c8be5600/cffi-2.0.0-cp39-cp39-manylinux2014_s390x.manylinux_2_17_s390x.whl", hash = "sha256:cb527a79772e5ef98fb1d700678fe031e353e765d1ca2d409c92263c6d43e09f", size = 203158, upload-time = "2025-09-08T23:23:55.169Z" },
 180      { url = "https://files.pythonhosted.org/packages/1f/74/cc4096ce66f5939042ae094e2e96f53426a979864aa1f96a621ad128be27/cffi-2.0.0-cp39-cp39-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:61d028e90346df14fedc3d1e5441df818d095f3b87d286825dfcbd6459b7ef63", size = 216548, upload-time = "2025-09-08T23:23:56.506Z" },
 181      { url = "https://files.pythonhosted.org/packages/e8/be/f6424d1dc46b1091ffcc8964fa7c0ab0cd36839dd2761b49c90481a6ba1b/cffi-2.0.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:0f6084a0ea23d05d20c3edcda20c3d006f9b6f3fefeac38f59262e10cef47ee2", size = 218897, upload-time = "2025-09-08T23:23:57.825Z" },
 182      { url = "https://files.pythonhosted.org/packages/f7/e0/dda537c2309817edf60109e39265f24f24aa7f050767e22c98c53fe7f48b/cffi-2.0.0-cp39-cp39-musllinux_1_2_i686.whl", hash = "sha256:1cd13c99ce269b3ed80b417dcd591415d3372bcac067009b6e0f59c7d4015e65", size = 211249, upload-time = "2025-09-08T23:23:59.139Z" },
 183      { url = "https://files.pythonhosted.org/packages/2b/e7/7c769804eb75e4c4b35e658dba01de1640a351a9653c3d49ca89d16ccc91/cffi-2.0.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:89472c9762729b5ae1ad974b777416bfda4ac5642423fa93bd57a09204712322", size = 218041, upload-time = "2025-09-08T23:24:00.496Z" },
 184      { url = "https://files.pythonhosted.org/packages/aa/d9/6218d78f920dcd7507fc16a766b5ef8f3b913cc7aa938e7fc80b9978d089/cffi-2.0.0-cp39-cp39-win32.whl", hash = "sha256:2081580ebb843f759b9f617314a24ed5738c51d2aee65d31e02f6f7a2b97707a", size = 172138, upload-time = "2025-09-08T23:24:01.7Z" },
 185      { url = "https://files.pythonhosted.org/packages/54/8f/a1e836f82d8e32a97e6b29cc8f641779181ac7363734f12df27db803ebda/cffi-2.0.0-cp39-cp39-win_amd64.whl", hash = "sha256:b882b3df248017dba09d6b16defe9b5c407fe32fc7c65a9c69798e6175601be9", size = 182794, upload-time = "2025-09-08T23:24:02.943Z" },
 186  ]
 187  
 188  [[package]]
 189  name = "charset-normalizer"
 190  version = "3.4.4"
 191  source = { registry = "https://pypi.org/simple" }
 192  sdist = { url = "https://files.pythonhosted.org/packages/13/69/33ddede1939fdd074bce5434295f38fae7136463422fe4fd3e0e89b98062/charset_normalizer-3.4.4.tar.gz", hash = "sha256:94537985111c35f28720e43603b8e7b43a6ecfb2ce1d3058bbe955b73404e21a", size = 129418, upload-time = "2025-10-14T04:42:32.879Z" }
 193  wheels = [
 194      { url = "https://files.pythonhosted.org/packages/1f/b8/6d51fc1d52cbd52cd4ccedd5b5b2f0f6a11bbf6765c782298b0f3e808541/charset_normalizer-3.4.4-cp310-cp310-macosx_10_9_universal2.whl", hash = "sha256:e824f1492727fa856dd6eda4f7cee25f8518a12f3c4a56a74e8095695089cf6d", size = 209709, upload-time = "2025-10-14T04:40:11.385Z" },
 195      { url = "https://files.pythonhosted.org/packages/5c/af/1f9d7f7faafe2ddfb6f72a2e07a548a629c61ad510fe60f9630309908fef/charset_normalizer-3.4.4-cp310-cp310-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:4bd5d4137d500351a30687c2d3971758aac9a19208fc110ccb9d7188fbe709e8", size = 148814, upload-time = "2025-10-14T04:40:13.135Z" },
 196      { url = "https://files.pythonhosted.org/packages/79/3d/f2e3ac2bbc056ca0c204298ea4e3d9db9b4afe437812638759db2c976b5f/charset_normalizer-3.4.4-cp310-cp310-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:027f6de494925c0ab2a55eab46ae5129951638a49a34d87f4c3eda90f696b4ad", size = 144467, upload-time = "2025-10-14T04:40:14.728Z" },
 197      { url = "https://files.pythonhosted.org/packages/ec/85/1bf997003815e60d57de7bd972c57dc6950446a3e4ccac43bc3070721856/charset_normalizer-3.4.4-cp310-cp310-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:f820802628d2694cb7e56db99213f930856014862f3fd943d290ea8438d07ca8", size = 162280, upload-time = "2025-10-14T04:40:16.14Z" },
 198      { url = "https://files.pythonhosted.org/packages/3e/8e/6aa1952f56b192f54921c436b87f2aaf7c7a7c3d0d1a765547d64fd83c13/charset_normalizer-3.4.4-cp310-cp310-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:798d75d81754988d2565bff1b97ba5a44411867c0cf32b77a7e8f8d84796b10d", size = 159454, upload-time = "2025-10-14T04:40:17.567Z" },
 199      { url = "https://files.pythonhosted.org/packages/36/3b/60cbd1f8e93aa25d1c669c649b7a655b0b5fb4c571858910ea9332678558/charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:9d1bb833febdff5c8927f922386db610b49db6e0d4f4ee29601d71e7c2694313", size = 153609, upload-time = "2025-10-14T04:40:19.08Z" },
 200      { url = "https://files.pythonhosted.org/packages/64/91/6a13396948b8fd3c4b4fd5bc74d045f5637d78c9675585e8e9fbe5636554/charset_normalizer-3.4.4-cp310-cp310-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:9cd98cdc06614a2f768d2b7286d66805f94c48cde050acdbbb7db2600ab3197e", size = 151849, upload-time = "2025-10-14T04:40:20.607Z" },
 201      { url = "https://files.pythonhosted.org/packages/b7/7a/59482e28b9981d105691e968c544cc0df3b7d6133152fb3dcdc8f135da7a/charset_normalizer-3.4.4-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:077fbb858e903c73f6c9db43374fd213b0b6a778106bc7032446a8e8b5b38b93", size = 151586, upload-time = "2025-10-14T04:40:21.719Z" },
 202      { url = "https://files.pythonhosted.org/packages/92/59/f64ef6a1c4bdd2baf892b04cd78792ed8684fbc48d4c2afe467d96b4df57/charset_normalizer-3.4.4-cp310-cp310-musllinux_1_2_armv7l.whl", hash = "sha256:244bfb999c71b35de57821b8ea746b24e863398194a4014e4c76adc2bbdfeff0", size = 145290, upload-time = "2025-10-14T04:40:23.069Z" },
 203      { url = "https://files.pythonhosted.org/packages/6b/63/3bf9f279ddfa641ffa1962b0db6a57a9c294361cc2f5fcac997049a00e9c/charset_normalizer-3.4.4-cp310-cp310-musllinux_1_2_ppc64le.whl", hash = "sha256:64b55f9dce520635f018f907ff1b0df1fdc31f2795a922fb49dd14fbcdf48c84", size = 163663, upload-time = "2025-10-14T04:40:24.17Z" },
 204      { url = "https://files.pythonhosted.org/packages/ed/09/c9e38fc8fa9e0849b172b581fd9803bdf6e694041127933934184e19f8c3/charset_normalizer-3.4.4-cp310-cp310-musllinux_1_2_riscv64.whl", hash = "sha256:faa3a41b2b66b6e50f84ae4a68c64fcd0c44355741c6374813a800cd6695db9e", size = 151964, upload-time = "2025-10-14T04:40:25.368Z" },
 205      { url = "https://files.pythonhosted.org/packages/d2/d1/d28b747e512d0da79d8b6a1ac18b7ab2ecfd81b2944c4c710e166d8dd09c/charset_normalizer-3.4.4-cp310-cp310-musllinux_1_2_s390x.whl", hash = "sha256:6515f3182dbe4ea06ced2d9e8666d97b46ef4c75e326b79bb624110f122551db", size = 161064, upload-time = "2025-10-14T04:40:26.806Z" },
 206      { url = "https://files.pythonhosted.org/packages/bb/9a/31d62b611d901c3b9e5500c36aab0ff5eb442043fb3a1c254200d3d397d9/charset_normalizer-3.4.4-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:cc00f04ed596e9dc0da42ed17ac5e596c6ccba999ba6bd92b0e0aef2f170f2d6", size = 155015, upload-time = "2025-10-14T04:40:28.284Z" },
 207      { url = "https://files.pythonhosted.org/packages/1f/f3/107e008fa2bff0c8b9319584174418e5e5285fef32f79d8ee6a430d0039c/charset_normalizer-3.4.4-cp310-cp310-win32.whl", hash = "sha256:f34be2938726fc13801220747472850852fe6b1ea75869a048d6f896838c896f", size = 99792, upload-time = "2025-10-14T04:40:29.613Z" },
 208      { url = "https://files.pythonhosted.org/packages/eb/66/e396e8a408843337d7315bab30dbf106c38966f1819f123257f5520f8a96/charset_normalizer-3.4.4-cp310-cp310-win_amd64.whl", hash = "sha256:a61900df84c667873b292c3de315a786dd8dac506704dea57bc957bd31e22c7d", size = 107198, upload-time = "2025-10-14T04:40:30.644Z" },
 209      { url = "https://files.pythonhosted.org/packages/b5/58/01b4f815bf0312704c267f2ccb6e5d42bcc7752340cd487bc9f8c3710597/charset_normalizer-3.4.4-cp310-cp310-win_arm64.whl", hash = "sha256:cead0978fc57397645f12578bfd2d5ea9138ea0fac82b2f63f7f7c6877986a69", size = 100262, upload-time = "2025-10-14T04:40:32.108Z" },
 210      { url = "https://files.pythonhosted.org/packages/ed/27/c6491ff4954e58a10f69ad90aca8a1b6fe9c5d3c6f380907af3c37435b59/charset_normalizer-3.4.4-cp311-cp311-macosx_10_9_universal2.whl", hash = "sha256:6e1fcf0720908f200cd21aa4e6750a48ff6ce4afe7ff5a79a90d5ed8a08296f8", size = 206988, upload-time = "2025-10-14T04:40:33.79Z" },
 211      { url = "https://files.pythonhosted.org/packages/94/59/2e87300fe67ab820b5428580a53cad894272dbb97f38a7a814a2a1ac1011/charset_normalizer-3.4.4-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:5f819d5fe9234f9f82d75bdfa9aef3a3d72c4d24a6e57aeaebba32a704553aa0", size = 147324, upload-time = "2025-10-14T04:40:34.961Z" },
 212      { url = "https://files.pythonhosted.org/packages/07/fb/0cf61dc84b2b088391830f6274cb57c82e4da8bbc2efeac8c025edb88772/charset_normalizer-3.4.4-cp311-cp311-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:a59cb51917aa591b1c4e6a43c132f0cdc3c76dbad6155df4e28ee626cc77a0a3", size = 142742, upload-time = "2025-10-14T04:40:36.105Z" },
 213      { url = "https://files.pythonhosted.org/packages/62/8b/171935adf2312cd745d290ed93cf16cf0dfe320863ab7cbeeae1dcd6535f/charset_normalizer-3.4.4-cp311-cp311-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:8ef3c867360f88ac904fd3f5e1f902f13307af9052646963ee08ff4f131adafc", size = 160863, upload-time = "2025-10-14T04:40:37.188Z" },
 214      { url = "https://files.pythonhosted.org/packages/09/73/ad875b192bda14f2173bfc1bc9a55e009808484a4b256748d931b6948442/charset_normalizer-3.4.4-cp311-cp311-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:d9e45d7faa48ee908174d8fe84854479ef838fc6a705c9315372eacbc2f02897", size = 157837, upload-time = "2025-10-14T04:40:38.435Z" },
 215      { url = "https://files.pythonhosted.org/packages/6d/fc/de9cce525b2c5b94b47c70a4b4fb19f871b24995c728e957ee68ab1671ea/charset_normalizer-3.4.4-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:840c25fb618a231545cbab0564a799f101b63b9901f2569faecd6b222ac72381", size = 151550, upload-time = "2025-10-14T04:40:40.053Z" },
 216      { url = "https://files.pythonhosted.org/packages/55/c2/43edd615fdfba8c6f2dfbd459b25a6b3b551f24ea21981e23fb768503ce1/charset_normalizer-3.4.4-cp311-cp311-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:ca5862d5b3928c4940729dacc329aa9102900382fea192fc5e52eb69d6093815", size = 149162, upload-time = "2025-10-14T04:40:41.163Z" },
 217      { url = "https://files.pythonhosted.org/packages/03/86/bde4ad8b4d0e9429a4e82c1e8f5c659993a9a863ad62c7df05cf7b678d75/charset_normalizer-3.4.4-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:d9c7f57c3d666a53421049053eaacdd14bbd0a528e2186fcb2e672effd053bb0", size = 150019, upload-time = "2025-10-14T04:40:42.276Z" },
 218      { url = "https://files.pythonhosted.org/packages/1f/86/a151eb2af293a7e7bac3a739b81072585ce36ccfb4493039f49f1d3cae8c/charset_normalizer-3.4.4-cp311-cp311-musllinux_1_2_armv7l.whl", hash = "sha256:277e970e750505ed74c832b4bf75dac7476262ee2a013f5574dd49075879e161", size = 143310, upload-time = "2025-10-14T04:40:43.439Z" },
 219      { url = "https://files.pythonhosted.org/packages/b5/fe/43dae6144a7e07b87478fdfc4dbe9efd5defb0e7ec29f5f58a55aeef7bf7/charset_normalizer-3.4.4-cp311-cp311-musllinux_1_2_ppc64le.whl", hash = "sha256:31fd66405eaf47bb62e8cd575dc621c56c668f27d46a61d975a249930dd5e2a4", size = 162022, upload-time = "2025-10-14T04:40:44.547Z" },
 220      { url = "https://files.pythonhosted.org/packages/80/e6/7aab83774f5d2bca81f42ac58d04caf44f0cc2b65fc6db2b3b2e8a05f3b3/charset_normalizer-3.4.4-cp311-cp311-musllinux_1_2_riscv64.whl", hash = "sha256:0d3d8f15c07f86e9ff82319b3d9ef6f4bf907608f53fe9d92b28ea9ae3d1fd89", size = 149383, upload-time = "2025-10-14T04:40:46.018Z" },
 221      { url = "https://files.pythonhosted.org/packages/4f/e8/b289173b4edae05c0dde07f69f8db476a0b511eac556dfe0d6bda3c43384/charset_normalizer-3.4.4-cp311-cp311-musllinux_1_2_s390x.whl", hash = "sha256:9f7fcd74d410a36883701fafa2482a6af2ff5ba96b9a620e9e0721e28ead5569", size = 159098, upload-time = "2025-10-14T04:40:47.081Z" },
 222      { url = "https://files.pythonhosted.org/packages/d8/df/fe699727754cae3f8478493c7f45f777b17c3ef0600e28abfec8619eb49c/charset_normalizer-3.4.4-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:ebf3e58c7ec8a8bed6d66a75d7fb37b55e5015b03ceae72a8e7c74495551e224", size = 152991, upload-time = "2025-10-14T04:40:48.246Z" },
 223      { url = "https://files.pythonhosted.org/packages/1a/86/584869fe4ddb6ffa3bd9f491b87a01568797fb9bd8933f557dba9771beaf/charset_normalizer-3.4.4-cp311-cp311-win32.whl", hash = "sha256:eecbc200c7fd5ddb9a7f16c7decb07b566c29fa2161a16cf67b8d068bd21690a", size = 99456, upload-time = "2025-10-14T04:40:49.376Z" },
 224      { url = "https://files.pythonhosted.org/packages/65/f6/62fdd5feb60530f50f7e38b4f6a1d5203f4d16ff4f9f0952962c044e919a/charset_normalizer-3.4.4-cp311-cp311-win_amd64.whl", hash = "sha256:5ae497466c7901d54b639cf42d5b8c1b6a4fead55215500d2f486d34db48d016", size = 106978, upload-time = "2025-10-14T04:40:50.844Z" },
 225      { url = "https://files.pythonhosted.org/packages/7a/9d/0710916e6c82948b3be62d9d398cb4fcf4e97b56d6a6aeccd66c4b2f2bd5/charset_normalizer-3.4.4-cp311-cp311-win_arm64.whl", hash = "sha256:65e2befcd84bc6f37095f5961e68a6f077bf44946771354a28ad434c2cce0ae1", size = 99969, upload-time = "2025-10-14T04:40:52.272Z" },
 226      { url = "https://files.pythonhosted.org/packages/f3/85/1637cd4af66fa687396e757dec650f28025f2a2f5a5531a3208dc0ec43f2/charset_normalizer-3.4.4-cp312-cp312-macosx_10_13_universal2.whl", hash = "sha256:0a98e6759f854bd25a58a73fa88833fba3b7c491169f86ce1180c948ab3fd394", size = 208425, upload-time = "2025-10-14T04:40:53.353Z" },
 227      { url = "https://files.pythonhosted.org/packages/9d/6a/04130023fef2a0d9c62d0bae2649b69f7b7d8d24ea5536feef50551029df/charset_normalizer-3.4.4-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:b5b290ccc2a263e8d185130284f8501e3e36c5e02750fc6b6bdeb2e9e96f1e25", size = 148162, upload-time = "2025-10-14T04:40:54.558Z" },
 228      { url = "https://files.pythonhosted.org/packages/78/29/62328d79aa60da22c9e0b9a66539feae06ca0f5a4171ac4f7dc285b83688/charset_normalizer-3.4.4-cp312-cp312-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:74bb723680f9f7a6234dcf67aea57e708ec1fbdf5699fb91dfd6f511b0a320ef", size = 144558, upload-time = "2025-10-14T04:40:55.677Z" },
 229      { url = "https://files.pythonhosted.org/packages/86/bb/b32194a4bf15b88403537c2e120b817c61cd4ecffa9b6876e941c3ee38fe/charset_normalizer-3.4.4-cp312-cp312-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:f1e34719c6ed0b92f418c7c780480b26b5d9c50349e9a9af7d76bf757530350d", size = 161497, upload-time = "2025-10-14T04:40:57.217Z" },
 230      { url = "https://files.pythonhosted.org/packages/19/89/a54c82b253d5b9b111dc74aca196ba5ccfcca8242d0fb64146d4d3183ff1/charset_normalizer-3.4.4-cp312-cp312-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:2437418e20515acec67d86e12bf70056a33abdacb5cb1655042f6538d6b085a8", size = 159240, upload-time = "2025-10-14T04:40:58.358Z" },
 231      { url = "https://files.pythonhosted.org/packages/c0/10/d20b513afe03acc89ec33948320a5544d31f21b05368436d580dec4e234d/charset_normalizer-3.4.4-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:11d694519d7f29d6cd09f6ac70028dba10f92f6cdd059096db198c283794ac86", size = 153471, upload-time = "2025-10-14T04:40:59.468Z" },
 232      { url = "https://files.pythonhosted.org/packages/61/fa/fbf177b55bdd727010f9c0a3c49eefa1d10f960e5f09d1d887bf93c2e698/charset_normalizer-3.4.4-cp312-cp312-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:ac1c4a689edcc530fc9d9aa11f5774b9e2f33f9a0c6a57864e90908f5208d30a", size = 150864, upload-time = "2025-10-14T04:41:00.623Z" },
 233      { url = "https://files.pythonhosted.org/packages/05/12/9fbc6a4d39c0198adeebbde20b619790e9236557ca59fc40e0e3cebe6f40/charset_normalizer-3.4.4-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:21d142cc6c0ec30d2efee5068ca36c128a30b0f2c53c1c07bd78cb6bc1d3be5f", size = 150647, upload-time = "2025-10-14T04:41:01.754Z" },
 234      { url = "https://files.pythonhosted.org/packages/ad/1f/6a9a593d52e3e8c5d2b167daf8c6b968808efb57ef4c210acb907c365bc4/charset_normalizer-3.4.4-cp312-cp312-musllinux_1_2_armv7l.whl", hash = "sha256:5dbe56a36425d26d6cfb40ce79c314a2e4dd6211d51d6d2191c00bed34f354cc", size = 145110, upload-time = "2025-10-14T04:41:03.231Z" },
 235      { url = "https://files.pythonhosted.org/packages/30/42/9a52c609e72471b0fc54386dc63c3781a387bb4fe61c20231a4ebcd58bdd/charset_normalizer-3.4.4-cp312-cp312-musllinux_1_2_ppc64le.whl", hash = "sha256:5bfbb1b9acf3334612667b61bd3002196fe2a1eb4dd74d247e0f2a4d50ec9bbf", size = 162839, upload-time = "2025-10-14T04:41:04.715Z" },
 236      { url = "https://files.pythonhosted.org/packages/c4/5b/c0682bbf9f11597073052628ddd38344a3d673fda35a36773f7d19344b23/charset_normalizer-3.4.4-cp312-cp312-musllinux_1_2_riscv64.whl", hash = "sha256:d055ec1e26e441f6187acf818b73564e6e6282709e9bcb5b63f5b23068356a15", size = 150667, upload-time = "2025-10-14T04:41:05.827Z" },
 237      { url = "https://files.pythonhosted.org/packages/e4/24/a41afeab6f990cf2daf6cb8c67419b63b48cf518e4f56022230840c9bfb2/charset_normalizer-3.4.4-cp312-cp312-musllinux_1_2_s390x.whl", hash = "sha256:af2d8c67d8e573d6de5bc30cdb27e9b95e49115cd9baad5ddbd1a6207aaa82a9", size = 160535, upload-time = "2025-10-14T04:41:06.938Z" },
 238      { url = "https://files.pythonhosted.org/packages/2a/e5/6a4ce77ed243c4a50a1fecca6aaaab419628c818a49434be428fe24c9957/charset_normalizer-3.4.4-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:780236ac706e66881f3b7f2f32dfe90507a09e67d1d454c762cf642e6e1586e0", size = 154816, upload-time = "2025-10-14T04:41:08.101Z" },
 239      { url = "https://files.pythonhosted.org/packages/a8/ef/89297262b8092b312d29cdb2517cb1237e51db8ecef2e9af5edbe7b683b1/charset_normalizer-3.4.4-cp312-cp312-win32.whl", hash = "sha256:5833d2c39d8896e4e19b689ffc198f08ea58116bee26dea51e362ecc7cd3ed26", size = 99694, upload-time = "2025-10-14T04:41:09.23Z" },
 240      { url = "https://files.pythonhosted.org/packages/3d/2d/1e5ed9dd3b3803994c155cd9aacb60c82c331bad84daf75bcb9c91b3295e/charset_normalizer-3.4.4-cp312-cp312-win_amd64.whl", hash = "sha256:a79cfe37875f822425b89a82333404539ae63dbdddf97f84dcbc3d339aae9525", size = 107131, upload-time = "2025-10-14T04:41:10.467Z" },
 241      { url = "https://files.pythonhosted.org/packages/d0/d9/0ed4c7098a861482a7b6a95603edce4c0d9db2311af23da1fb2b75ec26fc/charset_normalizer-3.4.4-cp312-cp312-win_arm64.whl", hash = "sha256:376bec83a63b8021bb5c8ea75e21c4ccb86e7e45ca4eb81146091b56599b80c3", size = 100390, upload-time = "2025-10-14T04:41:11.915Z" },
 242      { url = "https://files.pythonhosted.org/packages/97/45/4b3a1239bbacd321068ea6e7ac28875b03ab8bc0aa0966452db17cd36714/charset_normalizer-3.4.4-cp313-cp313-macosx_10_13_universal2.whl", hash = "sha256:e1f185f86a6f3403aa2420e815904c67b2f9ebc443f045edd0de921108345794", size = 208091, upload-time = "2025-10-14T04:41:13.346Z" },
 243      { url = "https://files.pythonhosted.org/packages/7d/62/73a6d7450829655a35bb88a88fca7d736f9882a27eacdca2c6d505b57e2e/charset_normalizer-3.4.4-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:6b39f987ae8ccdf0d2642338faf2abb1862340facc796048b604ef14919e55ed", size = 147936, upload-time = "2025-10-14T04:41:14.461Z" },
 244      { url = "https://files.pythonhosted.org/packages/89/c5/adb8c8b3d6625bef6d88b251bbb0d95f8205831b987631ab0c8bb5d937c2/charset_normalizer-3.4.4-cp313-cp313-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:3162d5d8ce1bb98dd51af660f2121c55d0fa541b46dff7bb9b9f86ea1d87de72", size = 144180, upload-time = "2025-10-14T04:41:15.588Z" },
 245      { url = "https://files.pythonhosted.org/packages/91/ed/9706e4070682d1cc219050b6048bfd293ccf67b3d4f5a4f39207453d4b99/charset_normalizer-3.4.4-cp313-cp313-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:81d5eb2a312700f4ecaa977a8235b634ce853200e828fbadf3a9c50bab278328", size = 161346, upload-time = "2025-10-14T04:41:16.738Z" },
 246      { url = "https://files.pythonhosted.org/packages/d5/0d/031f0d95e4972901a2f6f09ef055751805ff541511dc1252ba3ca1f80cf5/charset_normalizer-3.4.4-cp313-cp313-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:5bd2293095d766545ec1a8f612559f6b40abc0eb18bb2f5d1171872d34036ede", size = 158874, upload-time = "2025-10-14T04:41:17.923Z" },
 247      { url = "https://files.pythonhosted.org/packages/f5/83/6ab5883f57c9c801ce5e5677242328aa45592be8a00644310a008d04f922/charset_normalizer-3.4.4-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:a8a8b89589086a25749f471e6a900d3f662d1d3b6e2e59dcecf787b1cc3a1894", size = 153076, upload-time = "2025-10-14T04:41:19.106Z" },
 248      { url = "https://files.pythonhosted.org/packages/75/1e/5ff781ddf5260e387d6419959ee89ef13878229732732ee73cdae01800f2/charset_normalizer-3.4.4-cp313-cp313-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:bc7637e2f80d8530ee4a78e878bce464f70087ce73cf7c1caf142416923b98f1", size = 150601, upload-time = "2025-10-14T04:41:20.245Z" },
 249      { url = "https://files.pythonhosted.org/packages/d7/57/71be810965493d3510a6ca79b90c19e48696fb1ff964da319334b12677f0/charset_normalizer-3.4.4-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:f8bf04158c6b607d747e93949aa60618b61312fe647a6369f88ce2ff16043490", size = 150376, upload-time = "2025-10-14T04:41:21.398Z" },
 250      { url = "https://files.pythonhosted.org/packages/e5/d5/c3d057a78c181d007014feb7e9f2e65905a6c4ef182c0ddf0de2924edd65/charset_normalizer-3.4.4-cp313-cp313-musllinux_1_2_armv7l.whl", hash = "sha256:554af85e960429cf30784dd47447d5125aaa3b99a6f0683589dbd27e2f45da44", size = 144825, upload-time = "2025-10-14T04:41:22.583Z" },
 251      { url = "https://files.pythonhosted.org/packages/e6/8c/d0406294828d4976f275ffbe66f00266c4b3136b7506941d87c00cab5272/charset_normalizer-3.4.4-cp313-cp313-musllinux_1_2_ppc64le.whl", hash = "sha256:74018750915ee7ad843a774364e13a3db91682f26142baddf775342c3f5b1133", size = 162583, upload-time = "2025-10-14T04:41:23.754Z" },
 252      { url = "https://files.pythonhosted.org/packages/d7/24/e2aa1f18c8f15c4c0e932d9287b8609dd30ad56dbe41d926bd846e22fb8d/charset_normalizer-3.4.4-cp313-cp313-musllinux_1_2_riscv64.whl", hash = "sha256:c0463276121fdee9c49b98908b3a89c39be45d86d1dbaa22957e38f6321d4ce3", size = 150366, upload-time = "2025-10-14T04:41:25.27Z" },
 253      { url = "https://files.pythonhosted.org/packages/e4/5b/1e6160c7739aad1e2df054300cc618b06bf784a7a164b0f238360721ab86/charset_normalizer-3.4.4-cp313-cp313-musllinux_1_2_s390x.whl", hash = "sha256:362d61fd13843997c1c446760ef36f240cf81d3ebf74ac62652aebaf7838561e", size = 160300, upload-time = "2025-10-14T04:41:26.725Z" },
 254      { url = "https://files.pythonhosted.org/packages/7a/10/f882167cd207fbdd743e55534d5d9620e095089d176d55cb22d5322f2afd/charset_normalizer-3.4.4-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:9a26f18905b8dd5d685d6d07b0cdf98a79f3c7a918906af7cc143ea2e164c8bc", size = 154465, upload-time = "2025-10-14T04:41:28.322Z" },
 255      { url = "https://files.pythonhosted.org/packages/89/66/c7a9e1b7429be72123441bfdbaf2bc13faab3f90b933f664db506dea5915/charset_normalizer-3.4.4-cp313-cp313-win32.whl", hash = "sha256:9b35f4c90079ff2e2edc5b26c0c77925e5d2d255c42c74fdb70fb49b172726ac", size = 99404, upload-time = "2025-10-14T04:41:29.95Z" },
 256      { url = "https://files.pythonhosted.org/packages/c4/26/b9924fa27db384bdcd97ab83b4f0a8058d96ad9626ead570674d5e737d90/charset_normalizer-3.4.4-cp313-cp313-win_amd64.whl", hash = "sha256:b435cba5f4f750aa6c0a0d92c541fb79f69a387c91e61f1795227e4ed9cece14", size = 107092, upload-time = "2025-10-14T04:41:31.188Z" },
 257      { url = "https://files.pythonhosted.org/packages/af/8f/3ed4bfa0c0c72a7ca17f0380cd9e4dd842b09f664e780c13cff1dcf2ef1b/charset_normalizer-3.4.4-cp313-cp313-win_arm64.whl", hash = "sha256:542d2cee80be6f80247095cc36c418f7bddd14f4a6de45af91dfad36d817bba2", size = 100408, upload-time = "2025-10-14T04:41:32.624Z" },
 258      { url = "https://files.pythonhosted.org/packages/2a/35/7051599bd493e62411d6ede36fd5af83a38f37c4767b92884df7301db25d/charset_normalizer-3.4.4-cp314-cp314-macosx_10_13_universal2.whl", hash = "sha256:da3326d9e65ef63a817ecbcc0df6e94463713b754fe293eaa03da99befb9a5bd", size = 207746, upload-time = "2025-10-14T04:41:33.773Z" },
 259      { url = "https://files.pythonhosted.org/packages/10/9a/97c8d48ef10d6cd4fcead2415523221624bf58bcf68a802721a6bc807c8f/charset_normalizer-3.4.4-cp314-cp314-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:8af65f14dc14a79b924524b1e7fffe304517b2bff5a58bf64f30b98bbc5079eb", size = 147889, upload-time = "2025-10-14T04:41:34.897Z" },
 260      { url = "https://files.pythonhosted.org/packages/10/bf/979224a919a1b606c82bd2c5fa49b5c6d5727aa47b4312bb27b1734f53cd/charset_normalizer-3.4.4-cp314-cp314-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:74664978bb272435107de04e36db5a9735e78232b85b77d45cfb38f758efd33e", size = 143641, upload-time = "2025-10-14T04:41:36.116Z" },
 261      { url = "https://files.pythonhosted.org/packages/ba/33/0ad65587441fc730dc7bd90e9716b30b4702dc7b617e6ba4997dc8651495/charset_normalizer-3.4.4-cp314-cp314-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:752944c7ffbfdd10c074dc58ec2d5a8a4cd9493b314d367c14d24c17684ddd14", size = 160779, upload-time = "2025-10-14T04:41:37.229Z" },
 262      { url = "https://files.pythonhosted.org/packages/67/ed/331d6b249259ee71ddea93f6f2f0a56cfebd46938bde6fcc6f7b9a3d0e09/charset_normalizer-3.4.4-cp314-cp314-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:d1f13550535ad8cff21b8d757a3257963e951d96e20ec82ab44bc64aeb62a191", size = 159035, upload-time = "2025-10-14T04:41:38.368Z" },
 263      { url = "https://files.pythonhosted.org/packages/67/ff/f6b948ca32e4f2a4576aa129d8bed61f2e0543bf9f5f2b7fc3758ed005c9/charset_normalizer-3.4.4-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:ecaae4149d99b1c9e7b88bb03e3221956f68fd6d50be2ef061b2381b61d20838", size = 152542, upload-time = "2025-10-14T04:41:39.862Z" },
 264      { url = "https://files.pythonhosted.org/packages/16/85/276033dcbcc369eb176594de22728541a925b2632f9716428c851b149e83/charset_normalizer-3.4.4-cp314-cp314-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:cb6254dc36b47a990e59e1068afacdcd02958bdcce30bb50cc1700a8b9d624a6", size = 149524, upload-time = "2025-10-14T04:41:41.319Z" },
 265      { url = "https://files.pythonhosted.org/packages/9e/f2/6a2a1f722b6aba37050e626530a46a68f74e63683947a8acff92569f979a/charset_normalizer-3.4.4-cp314-cp314-musllinux_1_2_aarch64.whl", hash = "sha256:c8ae8a0f02f57a6e61203a31428fa1d677cbe50c93622b4149d5c0f319c1d19e", size = 150395, upload-time = "2025-10-14T04:41:42.539Z" },
 266      { url = "https://files.pythonhosted.org/packages/60/bb/2186cb2f2bbaea6338cad15ce23a67f9b0672929744381e28b0592676824/charset_normalizer-3.4.4-cp314-cp314-musllinux_1_2_armv7l.whl", hash = "sha256:47cc91b2f4dd2833fddaedd2893006b0106129d4b94fdb6af1f4ce5a9965577c", size = 143680, upload-time = "2025-10-14T04:41:43.661Z" },
 267      { url = "https://files.pythonhosted.org/packages/7d/a5/bf6f13b772fbb2a90360eb620d52ed8f796f3c5caee8398c3b2eb7b1c60d/charset_normalizer-3.4.4-cp314-cp314-musllinux_1_2_ppc64le.whl", hash = "sha256:82004af6c302b5d3ab2cfc4cc5f29db16123b1a8417f2e25f9066f91d4411090", size = 162045, upload-time = "2025-10-14T04:41:44.821Z" },
 268      { url = "https://files.pythonhosted.org/packages/df/c5/d1be898bf0dc3ef9030c3825e5d3b83f2c528d207d246cbabe245966808d/charset_normalizer-3.4.4-cp314-cp314-musllinux_1_2_riscv64.whl", hash = "sha256:2b7d8f6c26245217bd2ad053761201e9f9680f8ce52f0fcd8d0755aeae5b2152", size = 149687, upload-time = "2025-10-14T04:41:46.442Z" },
 269      { url = "https://files.pythonhosted.org/packages/a5/42/90c1f7b9341eef50c8a1cb3f098ac43b0508413f33affd762855f67a410e/charset_normalizer-3.4.4-cp314-cp314-musllinux_1_2_s390x.whl", hash = "sha256:799a7a5e4fb2d5898c60b640fd4981d6a25f1c11790935a44ce38c54e985f828", size = 160014, upload-time = "2025-10-14T04:41:47.631Z" },
 270      { url = "https://files.pythonhosted.org/packages/76/be/4d3ee471e8145d12795ab655ece37baed0929462a86e72372fd25859047c/charset_normalizer-3.4.4-cp314-cp314-musllinux_1_2_x86_64.whl", hash = "sha256:99ae2cffebb06e6c22bdc25801d7b30f503cc87dbd283479e7b606f70aff57ec", size = 154044, upload-time = "2025-10-14T04:41:48.81Z" },
 271      { url = "https://files.pythonhosted.org/packages/b0/6f/8f7af07237c34a1defe7defc565a9bc1807762f672c0fde711a4b22bf9c0/charset_normalizer-3.4.4-cp314-cp314-win32.whl", hash = "sha256:f9d332f8c2a2fcbffe1378594431458ddbef721c1769d78e2cbc06280d8155f9", size = 99940, upload-time = "2025-10-14T04:41:49.946Z" },
 272      { url = "https://files.pythonhosted.org/packages/4b/51/8ade005e5ca5b0d80fb4aff72a3775b325bdc3d27408c8113811a7cbe640/charset_normalizer-3.4.4-cp314-cp314-win_amd64.whl", hash = "sha256:8a6562c3700cce886c5be75ade4a5db4214fda19fede41d9792d100288d8f94c", size = 107104, upload-time = "2025-10-14T04:41:51.051Z" },
 273      { url = "https://files.pythonhosted.org/packages/da/5f/6b8f83a55bb8278772c5ae54a577f3099025f9ade59d0136ac24a0df4bde/charset_normalizer-3.4.4-cp314-cp314-win_arm64.whl", hash = "sha256:de00632ca48df9daf77a2c65a484531649261ec9f25489917f09e455cb09ddb2", size = 100743, upload-time = "2025-10-14T04:41:52.122Z" },
 274      { url = "https://files.pythonhosted.org/packages/0a/4e/3926a1c11f0433791985727965263f788af00db3482d89a7545ca5ecc921/charset_normalizer-3.4.4-cp38-cp38-macosx_10_9_universal2.whl", hash = "sha256:ce8a0633f41a967713a59c4139d29110c07e826d131a316b50ce11b1d79b4f84", size = 198599, upload-time = "2025-10-14T04:41:53.213Z" },
 275      { url = "https://files.pythonhosted.org/packages/ec/7c/b92d1d1dcffc34592e71ea19c882b6709e43d20fa498042dea8b815638d7/charset_normalizer-3.4.4-cp38-cp38-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:eaabd426fe94daf8fd157c32e571c85cb12e66692f15516a83a03264b08d06c3", size = 143090, upload-time = "2025-10-14T04:41:54.385Z" },
 276      { url = "https://files.pythonhosted.org/packages/84/ce/61a28d3bb77281eb24107b937a497f3c43089326d27832a63dcedaab0478/charset_normalizer-3.4.4-cp38-cp38-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:c4ef880e27901b6cc782f1b95f82da9313c0eb95c3af699103088fa0ac3ce9ac", size = 139490, upload-time = "2025-10-14T04:41:55.551Z" },
 277      { url = "https://files.pythonhosted.org/packages/c0/bd/c9e59a91b2061c6f8bb98a150670cb16d4cd7c4ba7d11ad0cdf789155f41/charset_normalizer-3.4.4-cp38-cp38-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:2aaba3b0819274cc41757a1da876f810a3e4d7b6eb25699253a4effef9e8e4af", size = 155334, upload-time = "2025-10-14T04:41:56.724Z" },
 278      { url = "https://files.pythonhosted.org/packages/bf/37/f17ae176a80f22ff823456af91ba3bc59df308154ff53aef0d39eb3d3419/charset_normalizer-3.4.4-cp38-cp38-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:778d2e08eda00f4256d7f672ca9fef386071c9202f5e4607920b86d7803387f2", size = 152823, upload-time = "2025-10-14T04:41:58.236Z" },
 279      { url = "https://files.pythonhosted.org/packages/bf/fa/cf5bb2409a385f78750e78c8d2e24780964976acdaaed65dbd6083ae5b40/charset_normalizer-3.4.4-cp38-cp38-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:f155a433c2ec037d4e8df17d18922c3a0d9b3232a396690f17175d2946f0218d", size = 147618, upload-time = "2025-10-14T04:41:59.409Z" },
 280      { url = "https://files.pythonhosted.org/packages/9b/63/579784a65bc7de2d4518d40bb8f1870900163e86f17f21fd1384318c459d/charset_normalizer-3.4.4-cp38-cp38-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:a8bf8d0f749c5757af2142fe7903a9df1d2e8aa3841559b2bad34b08d0e2bcf3", size = 145516, upload-time = "2025-10-14T04:42:00.579Z" },
 281      { url = "https://files.pythonhosted.org/packages/a3/a9/94ec6266cd394e8f93a4d69cca651d61bf6ac58d2a0422163b30c698f2c7/charset_normalizer-3.4.4-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:194f08cbb32dc406d6e1aea671a68be0823673db2832b38405deba2fb0d88f63", size = 145266, upload-time = "2025-10-14T04:42:01.684Z" },
 282      { url = "https://files.pythonhosted.org/packages/09/14/d6626eb97764b58c2779fa7928fa7d1a49adb8ce687c2dbba4db003c1939/charset_normalizer-3.4.4-cp38-cp38-musllinux_1_2_armv7l.whl", hash = "sha256:6aee717dcfead04c6eb1ce3bd29ac1e22663cdea57f943c87d1eab9a025438d7", size = 139559, upload-time = "2025-10-14T04:42:02.902Z" },
 283      { url = "https://files.pythonhosted.org/packages/09/01/ddbe6b01313ba191dbb0a43c7563bc770f2448c18127f9ea4b119c44dff0/charset_normalizer-3.4.4-cp38-cp38-musllinux_1_2_ppc64le.whl", hash = "sha256:cd4b7ca9984e5e7985c12bc60a6f173f3c958eae74f3ef6624bb6b26e2abbae4", size = 156653, upload-time = "2025-10-14T04:42:04.005Z" },
 284      { url = "https://files.pythonhosted.org/packages/95/c8/d05543378bea89296e9af4510b44c704626e191da447235c8fdedfc5b7b2/charset_normalizer-3.4.4-cp38-cp38-musllinux_1_2_riscv64.whl", hash = "sha256:b7cf1017d601aa35e6bb650b6ad28652c9cd78ee6caff19f3c28d03e1c80acbf", size = 145644, upload-time = "2025-10-14T04:42:05.211Z" },
 285      { url = "https://files.pythonhosted.org/packages/72/01/2866c4377998ef8a1f6802f6431e774a4c8ebe75b0a6e569ceec55c9cbfb/charset_normalizer-3.4.4-cp38-cp38-musllinux_1_2_s390x.whl", hash = "sha256:e912091979546adf63357d7e2ccff9b44f026c075aeaf25a52d0e95ad2281074", size = 153964, upload-time = "2025-10-14T04:42:06.341Z" },
 286      { url = "https://files.pythonhosted.org/packages/4a/66/66c72468a737b4cbd7851ba2c522fe35c600575fbeac944460b4fd4a06fe/charset_normalizer-3.4.4-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:5cb4d72eea50c8868f5288b7f7f33ed276118325c1dfd3957089f6b519e1382a", size = 148777, upload-time = "2025-10-14T04:42:07.535Z" },
 287      { url = "https://files.pythonhosted.org/packages/50/94/d0d56677fdddbffa8ca00ec411f67bb8c947f9876374ddc9d160d4f2c4b3/charset_normalizer-3.4.4-cp38-cp38-win32.whl", hash = "sha256:837c2ce8c5a65a2035be9b3569c684358dfbf109fd3b6969630a87535495ceaa", size = 98687, upload-time = "2025-10-14T04:42:08.678Z" },
 288      { url = "https://files.pythonhosted.org/packages/00/64/c3bc303d1b586480b1c8e6e1e2191a6d6dd40255244e5cf16763dcec52e6/charset_normalizer-3.4.4-cp38-cp38-win_amd64.whl", hash = "sha256:44c2a8734b333e0578090c4cd6b16f275e07aa6614ca8715e6c038e865e70576", size = 106115, upload-time = "2025-10-14T04:42:09.793Z" },
 289      { url = "https://files.pythonhosted.org/packages/46/7c/0c4760bccf082737ca7ab84a4c2034fcc06b1f21cf3032ea98bd6feb1725/charset_normalizer-3.4.4-cp39-cp39-macosx_10_9_universal2.whl", hash = "sha256:a9768c477b9d7bd54bc0c86dbaebdec6f03306675526c9927c0e8a04e8f94af9", size = 209609, upload-time = "2025-10-14T04:42:10.922Z" },
 290      { url = "https://files.pythonhosted.org/packages/bb/a4/69719daef2f3d7f1819de60c9a6be981b8eeead7542d5ec4440f3c80e111/charset_normalizer-3.4.4-cp39-cp39-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:1bee1e43c28aa63cb16e5c14e582580546b08e535299b8b6158a7c9c768a1f3d", size = 149029, upload-time = "2025-10-14T04:42:12.38Z" },
 291      { url = "https://files.pythonhosted.org/packages/e6/21/8d4e1d6c1e6070d3672908b8e4533a71b5b53e71d16828cc24d0efec564c/charset_normalizer-3.4.4-cp39-cp39-manylinux2014_armv7l.manylinux_2_17_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:fd44c878ea55ba351104cb93cc85e74916eb8fa440ca7903e57575e97394f608", size = 144580, upload-time = "2025-10-14T04:42:13.549Z" },
 292      { url = "https://files.pythonhosted.org/packages/a7/0a/a616d001b3f25647a9068e0b9199f697ce507ec898cacb06a0d5a1617c99/charset_normalizer-3.4.4-cp39-cp39-manylinux2014_ppc64le.manylinux_2_17_ppc64le.manylinux_2_28_ppc64le.whl", hash = "sha256:0f04b14ffe5fdc8c4933862d8306109a2c51e0704acfa35d51598eb45a1e89fc", size = 162340, upload-time = "2025-10-14T04:42:14.892Z" },
 293      { url = "https://files.pythonhosted.org/packages/85/93/060b52deb249a5450460e0585c88a904a83aec474ab8e7aba787f45e79f2/charset_normalizer-3.4.4-cp39-cp39-manylinux2014_s390x.manylinux_2_17_s390x.manylinux_2_28_s390x.whl", hash = "sha256:cd09d08005f958f370f539f186d10aec3377d55b9eeb0d796025d4886119d76e", size = 159619, upload-time = "2025-10-14T04:42:16.676Z" },
 294      { url = "https://files.pythonhosted.org/packages/dd/21/0274deb1cc0632cd587a9a0ec6b4674d9108e461cb4cd40d457adaeb0564/charset_normalizer-3.4.4-cp39-cp39-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:4fe7859a4e3e8457458e2ff592f15ccb02f3da787fcd31e0183879c3ad4692a1", size = 153980, upload-time = "2025-10-14T04:42:17.917Z" },
 295      { url = "https://files.pythonhosted.org/packages/28/2b/e3d7d982858dccc11b31906976323d790dded2017a0572f093ff982d692f/charset_normalizer-3.4.4-cp39-cp39-manylinux_2_31_riscv64.manylinux_2_39_riscv64.whl", hash = "sha256:fa09f53c465e532f4d3db095e0c55b615f010ad81803d383195b6b5ca6cbf5f3", size = 152174, upload-time = "2025-10-14T04:42:19.018Z" },
 296      { url = "https://files.pythonhosted.org/packages/6e/ff/4a269f8e35f1e58b2df52c131a1fa019acb7ef3f8697b7d464b07e9b492d/charset_normalizer-3.4.4-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:7fa17817dc5625de8a027cb8b26d9fefa3ea28c8253929b8d6649e705d2835b6", size = 151666, upload-time = "2025-10-14T04:42:20.171Z" },
 297      { url = "https://files.pythonhosted.org/packages/da/c9/ec39870f0b330d58486001dd8e532c6b9a905f5765f58a6f8204926b4a93/charset_normalizer-3.4.4-cp39-cp39-musllinux_1_2_armv7l.whl", hash = "sha256:5947809c8a2417be3267efc979c47d76a079758166f7d43ef5ae8e9f92751f88", size = 145550, upload-time = "2025-10-14T04:42:21.324Z" },
 298      { url = "https://files.pythonhosted.org/packages/75/8f/d186ab99e40e0ed9f82f033d6e49001701c81244d01905dd4a6924191a30/charset_normalizer-3.4.4-cp39-cp39-musllinux_1_2_ppc64le.whl", hash = "sha256:4902828217069c3c5c71094537a8e623f5d097858ac6ca8252f7b4d10b7560f1", size = 163721, upload-time = "2025-10-14T04:42:22.46Z" },
 299      { url = "https://files.pythonhosted.org/packages/96/b1/6047663b9744df26a7e479ac1e77af7134b1fcf9026243bb48ee2d18810f/charset_normalizer-3.4.4-cp39-cp39-musllinux_1_2_riscv64.whl", hash = "sha256:7c308f7e26e4363d79df40ca5b2be1c6ba9f02bdbccfed5abddb7859a6ce72cf", size = 152127, upload-time = "2025-10-14T04:42:23.712Z" },
 300      { url = "https://files.pythonhosted.org/packages/59/78/e5a6eac9179f24f704d1be67d08704c3c6ab9f00963963524be27c18ed87/charset_normalizer-3.4.4-cp39-cp39-musllinux_1_2_s390x.whl", hash = "sha256:2c9d3c380143a1fedbff95a312aa798578371eb29da42106a29019368a475318", size = 161175, upload-time = "2025-10-14T04:42:24.87Z" },
 301      { url = "https://files.pythonhosted.org/packages/e5/43/0e626e42d54dd2f8dd6fc5e1c5ff00f05fbca17cb699bedead2cae69c62f/charset_normalizer-3.4.4-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:cb01158d8b88ee68f15949894ccc6712278243d95f344770fa7593fa2d94410c", size = 155375, upload-time = "2025-10-14T04:42:27.246Z" },
 302      { url = "https://files.pythonhosted.org/packages/e9/91/d9615bf2e06f35e4997616ff31248c3657ed649c5ab9d35ea12fce54e380/charset_normalizer-3.4.4-cp39-cp39-win32.whl", hash = "sha256:2677acec1a2f8ef614c6888b5b4ae4060cc184174a938ed4e8ef690e15d3e505", size = 99692, upload-time = "2025-10-14T04:42:28.425Z" },
 303      { url = "https://files.pythonhosted.org/packages/d1/a9/6c040053909d9d1ef4fcab45fddec083aedc9052c10078339b47c8573ea8/charset_normalizer-3.4.4-cp39-cp39-win_amd64.whl", hash = "sha256:f8e160feb2aed042cd657a72acc0b481212ed28b1b9a95c0cee1621b524e1966", size = 107192, upload-time = "2025-10-14T04:42:29.482Z" },
 304      { url = "https://files.pythonhosted.org/packages/f0/c6/4fa536b2c0cd3edfb7ccf8469fa0f363ea67b7213a842b90909ca33dd851/charset_normalizer-3.4.4-cp39-cp39-win_arm64.whl", hash = "sha256:b5d84d37db046c5ca74ee7bb47dd6cbc13f80665fdde3e8040bdd3fb015ecb50", size = 100220, upload-time = "2025-10-14T04:42:30.632Z" },
 305      { url = "https://files.pythonhosted.org/packages/0a/4c/925909008ed5a988ccbb72dcc897407e5d6d3bd72410d69e051fc0c14647/charset_normalizer-3.4.4-py3-none-any.whl", hash = "sha256:7a32c560861a02ff789ad905a2fe94e3f840803362c84fecf1851cb4cf3dc37f", size = 53402, upload-time = "2025-10-14T04:42:31.76Z" },
 306  ]
 307  
 308  [[package]]
 309  name = "click"
 310  version = "8.1.8"
 311  source = { registry = "https://pypi.org/simple" }
 312  resolution-markers = [
 313      "python_full_version == '3.9.*'",
 314      "python_full_version < '3.9'",
 315  ]
 316  dependencies = [
 317      { name = "colorama", marker = "python_full_version < '3.10' and sys_platform == 'win32'" },
 318  ]
 319  sdist = { url = "https://files.pythonhosted.org/packages/b9/2e/0090cbf739cee7d23781ad4b89a9894a41538e4fcf4c31dcdd705b78eb8b/click-8.1.8.tar.gz", hash = "sha256:ed53c9d8990d83c2a27deae68e4ee337473f6330c040a31d4225c9574d16096a", size = 226593, upload-time = "2024-12-21T18:38:44.339Z" }
 320  wheels = [
 321      { url = "https://files.pythonhosted.org/packages/7e/d4/7ebdbd03970677812aac39c869717059dbb71a4cfc033ca6e5221787892c/click-8.1.8-py3-none-any.whl", hash = "sha256:63c132bbbed01578a06712a2d1f497bb62d9c1c0d329b7903a866228027263b2", size = 98188, upload-time = "2024-12-21T18:38:41.666Z" },
 322  ]
 323  
 324  [[package]]
 325  name = "click"
 326  version = "8.3.1"
 327  source = { registry = "https://pypi.org/simple" }
 328  resolution-markers = [
 329      "python_full_version >= '3.10'",
 330  ]
 331  dependencies = [
 332      { name = "colorama", marker = "python_full_version >= '3.10' and sys_platform == 'win32'" },
 333  ]
 334  sdist = { url = "https://files.pythonhosted.org/packages/3d/fa/656b739db8587d7b5dfa22e22ed02566950fbfbcdc20311993483657a5c0/click-8.3.1.tar.gz", hash = "sha256:12ff4785d337a1bb490bb7e9c2b1ee5da3112e94a8622f26a6c77f5d2fc6842a", size = 295065, upload-time = "2025-11-15T20:45:42.706Z" }
 335  wheels = [
 336      { url = "https://files.pythonhosted.org/packages/98/78/01c019cdb5d6498122777c1a43056ebb3ebfeef2076d9d026bfe15583b2b/click-8.3.1-py3-none-any.whl", hash = "sha256:981153a64e25f12d547d3426c367a4857371575ee7ad18df2a6183ab0545b2a6", size = 108274, upload-time = "2025-11-15T20:45:41.139Z" },
 337  ]
 338  
 339  [[package]]
 340  name = "colorama"
 341  version = "0.4.6"
 342  source = { registry = "https://pypi.org/simple" }
 343  sdist = { url = "https://files.pythonhosted.org/packages/d8/53/6f443c9a4a8358a93a6792e2acffb9d9d5cb0a5cfd8802644b7b1c9a02e4/colorama-0.4.6.tar.gz", hash = "sha256:08695f5cb7ed6e0531a20572697297273c47b8cae5a63ffc6d6ed5c201be6e44", size = 27697, upload-time = "2022-10-25T02:36:22.414Z" }
 344  wheels = [
 345      { url = "https://files.pythonhosted.org/packages/d1/d6/3965ed04c63042e047cb6a3e6ed1a63a35087b6a609aa3a15ed8ac56c221/colorama-0.4.6-py2.py3-none-any.whl", hash = "sha256:4f1d9991f5acc0ca119f9d443620b77f9d6b33703e51011c16baf57afb285fc6", size = 25335, upload-time = "2022-10-25T02:36:20.889Z" },
 346  ]
 347  
 348  [[package]]
 349  name = "cryptography"
 350  version = "46.0.3"
 351  source = { registry = "https://pypi.org/simple" }
 352  dependencies = [
 353      { name = "cffi", version = "1.17.1", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9' and platform_python_implementation != 'PyPy'" },
 354      { name = "cffi", version = "2.0.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.9' and platform_python_implementation != 'PyPy'" },
 355      { name = "typing-extensions", version = "4.13.2", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 356      { name = "typing-extensions", version = "4.15.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.9' and python_full_version < '3.11'" },
 357  ]
 358  sdist = { url = "https://files.pythonhosted.org/packages/9f/33/c00162f49c0e2fe8064a62cb92b93e50c74a72bc370ab92f86112b33ff62/cryptography-46.0.3.tar.gz", hash = "sha256:a8b17438104fed022ce745b362294d9ce35b4c2e45c1d958ad4a4b019285f4a1", size = 749258, upload-time = "2025-10-15T23:18:31.74Z" }
 359  wheels = [
 360      { url = "https://files.pythonhosted.org/packages/1d/42/9c391dd801d6cf0d561b5890549d4b27bafcc53b39c31a817e69d87c625b/cryptography-46.0.3-cp311-abi3-macosx_10_9_universal2.whl", hash = "sha256:109d4ddfadf17e8e7779c39f9b18111a09efb969a301a31e987416a0191ed93a", size = 7225004, upload-time = "2025-10-15T23:16:52.239Z" },
 361      { url = "https://files.pythonhosted.org/packages/1c/67/38769ca6b65f07461eb200e85fc1639b438bdc667be02cf7f2cd6a64601c/cryptography-46.0.3-cp311-abi3-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:09859af8466b69bc3c27bdf4f5d84a665e0f7ab5088412e9e2ec49758eca5cbc", size = 4296667, upload-time = "2025-10-15T23:16:54.369Z" },
 362      { url = "https://files.pythonhosted.org/packages/5c/49/498c86566a1d80e978b42f0d702795f69887005548c041636df6ae1ca64c/cryptography-46.0.3-cp311-abi3-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:01ca9ff2885f3acc98c29f1860552e37f6d7c7d013d7334ff2a9de43a449315d", size = 4450807, upload-time = "2025-10-15T23:16:56.414Z" },
 363      { url = "https://files.pythonhosted.org/packages/4b/0a/863a3604112174c8624a2ac3c038662d9e59970c7f926acdcfaed8d61142/cryptography-46.0.3-cp311-abi3-manylinux_2_28_aarch64.whl", hash = "sha256:6eae65d4c3d33da080cff9c4ab1f711b15c1d9760809dad6ea763f3812d254cb", size = 4299615, upload-time = "2025-10-15T23:16:58.442Z" },
 364      { url = "https://files.pythonhosted.org/packages/64/02/b73a533f6b64a69f3cd3872acb6ebc12aef924d8d103133bb3ea750dc703/cryptography-46.0.3-cp311-abi3-manylinux_2_28_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:e5bf0ed4490068a2e72ac03d786693adeb909981cc596425d09032d372bcc849", size = 4016800, upload-time = "2025-10-15T23:17:00.378Z" },
 365      { url = "https://files.pythonhosted.org/packages/25/d5/16e41afbfa450cde85a3b7ec599bebefaef16b5c6ba4ec49a3532336ed72/cryptography-46.0.3-cp311-abi3-manylinux_2_28_ppc64le.whl", hash = "sha256:5ecfccd2329e37e9b7112a888e76d9feca2347f12f37918facbb893d7bb88ee8", size = 4984707, upload-time = "2025-10-15T23:17:01.98Z" },
 366      { url = "https://files.pythonhosted.org/packages/c9/56/e7e69b427c3878352c2fb9b450bd0e19ed552753491d39d7d0a2f5226d41/cryptography-46.0.3-cp311-abi3-manylinux_2_28_x86_64.whl", hash = "sha256:a2c0cd47381a3229c403062f764160d57d4d175e022c1df84e168c6251a22eec", size = 4482541, upload-time = "2025-10-15T23:17:04.078Z" },
 367      { url = "https://files.pythonhosted.org/packages/78/f6/50736d40d97e8483172f1bb6e698895b92a223dba513b0ca6f06b2365339/cryptography-46.0.3-cp311-abi3-manylinux_2_34_aarch64.whl", hash = "sha256:549e234ff32571b1f4076ac269fcce7a808d3bf98b76c8dd560e42dbc66d7d91", size = 4299464, upload-time = "2025-10-15T23:17:05.483Z" },
 368      { url = "https://files.pythonhosted.org/packages/00/de/d8e26b1a855f19d9994a19c702fa2e93b0456beccbcfe437eda00e0701f2/cryptography-46.0.3-cp311-abi3-manylinux_2_34_ppc64le.whl", hash = "sha256:c0a7bb1a68a5d3471880e264621346c48665b3bf1c3759d682fc0864c540bd9e", size = 4950838, upload-time = "2025-10-15T23:17:07.425Z" },
 369      { url = "https://files.pythonhosted.org/packages/8f/29/798fc4ec461a1c9e9f735f2fc58741b0daae30688f41b2497dcbc9ed1355/cryptography-46.0.3-cp311-abi3-manylinux_2_34_x86_64.whl", hash = "sha256:10b01676fc208c3e6feeb25a8b83d81767e8059e1fe86e1dc62d10a3018fa926", size = 4481596, upload-time = "2025-10-15T23:17:09.343Z" },
 370      { url = "https://files.pythonhosted.org/packages/15/8d/03cd48b20a573adfff7652b76271078e3045b9f49387920e7f1f631d125e/cryptography-46.0.3-cp311-abi3-musllinux_1_2_aarch64.whl", hash = "sha256:0abf1ffd6e57c67e92af68330d05760b7b7efb243aab8377e583284dbab72c71", size = 4426782, upload-time = "2025-10-15T23:17:11.22Z" },
 371      { url = "https://files.pythonhosted.org/packages/fa/b1/ebacbfe53317d55cf33165bda24c86523497a6881f339f9aae5c2e13e57b/cryptography-46.0.3-cp311-abi3-musllinux_1_2_x86_64.whl", hash = "sha256:a04bee9ab6a4da801eb9b51f1b708a1b5b5c9eb48c03f74198464c66f0d344ac", size = 4698381, upload-time = "2025-10-15T23:17:12.829Z" },
 372      { url = "https://files.pythonhosted.org/packages/96/92/8a6a9525893325fc057a01f654d7efc2c64b9de90413adcf605a85744ff4/cryptography-46.0.3-cp311-abi3-win32.whl", hash = "sha256:f260d0d41e9b4da1ed1e0f1ce571f97fe370b152ab18778e9e8f67d6af432018", size = 3055988, upload-time = "2025-10-15T23:17:14.65Z" },
 373      { url = "https://files.pythonhosted.org/packages/7e/bf/80fbf45253ea585a1e492a6a17efcb93467701fa79e71550a430c5e60df0/cryptography-46.0.3-cp311-abi3-win_amd64.whl", hash = "sha256:a9a3008438615669153eb86b26b61e09993921ebdd75385ddd748702c5adfddb", size = 3514451, upload-time = "2025-10-15T23:17:16.142Z" },
 374      { url = "https://files.pythonhosted.org/packages/2e/af/9b302da4c87b0beb9db4e756386a7c6c5b8003cd0e742277888d352ae91d/cryptography-46.0.3-cp311-abi3-win_arm64.whl", hash = "sha256:5d7f93296ee28f68447397bf5198428c9aeeab45705a55d53a6343455dcb2c3c", size = 2928007, upload-time = "2025-10-15T23:17:18.04Z" },
 375      { url = "https://files.pythonhosted.org/packages/f5/e2/a510aa736755bffa9d2f75029c229111a1d02f8ecd5de03078f4c18d91a3/cryptography-46.0.3-cp314-cp314t-macosx_10_9_universal2.whl", hash = "sha256:00a5e7e87938e5ff9ff5447ab086a5706a957137e6e433841e9d24f38a065217", size = 7158012, upload-time = "2025-10-15T23:17:19.982Z" },
 376      { url = "https://files.pythonhosted.org/packages/73/dc/9aa866fbdbb95b02e7f9d086f1fccfeebf8953509b87e3f28fff927ff8a0/cryptography-46.0.3-cp314-cp314t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:c8daeb2d2174beb4575b77482320303f3d39b8e81153da4f0fb08eb5fe86a6c5", size = 4288728, upload-time = "2025-10-15T23:17:21.527Z" },
 377      { url = "https://files.pythonhosted.org/packages/c5/fd/bc1daf8230eaa075184cbbf5f8cd00ba9db4fd32d63fb83da4671b72ed8a/cryptography-46.0.3-cp314-cp314t-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:39b6755623145ad5eff1dab323f4eae2a32a77a7abef2c5089a04a3d04366715", size = 4435078, upload-time = "2025-10-15T23:17:23.042Z" },
 378      { url = "https://files.pythonhosted.org/packages/82/98/d3bd5407ce4c60017f8ff9e63ffee4200ab3e23fe05b765cab805a7db008/cryptography-46.0.3-cp314-cp314t-manylinux_2_28_aarch64.whl", hash = "sha256:db391fa7c66df6762ee3f00c95a89e6d428f4d60e7abc8328f4fe155b5ac6e54", size = 4293460, upload-time = "2025-10-15T23:17:24.885Z" },
 379      { url = "https://files.pythonhosted.org/packages/26/e9/e23e7900983c2b8af7a08098db406cf989d7f09caea7897e347598d4cd5b/cryptography-46.0.3-cp314-cp314t-manylinux_2_28_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:78a97cf6a8839a48c49271cdcbd5cf37ca2c1d6b7fdd86cc864f302b5e9bf459", size = 3995237, upload-time = "2025-10-15T23:17:26.449Z" },
 380      { url = "https://files.pythonhosted.org/packages/91/15/af68c509d4a138cfe299d0d7ddb14afba15233223ebd933b4bbdbc7155d3/cryptography-46.0.3-cp314-cp314t-manylinux_2_28_ppc64le.whl", hash = "sha256:dfb781ff7eaa91a6f7fd41776ec37c5853c795d3b358d4896fdbb5df168af422", size = 4967344, upload-time = "2025-10-15T23:17:28.06Z" },
 381      { url = "https://files.pythonhosted.org/packages/ca/e3/8643d077c53868b681af077edf6b3cb58288b5423610f21c62aadcbe99f4/cryptography-46.0.3-cp314-cp314t-manylinux_2_28_x86_64.whl", hash = "sha256:6f61efb26e76c45c4a227835ddeae96d83624fb0d29eb5df5b96e14ed1a0afb7", size = 4466564, upload-time = "2025-10-15T23:17:29.665Z" },
 382      { url = "https://files.pythonhosted.org/packages/0e/43/c1e8726fa59c236ff477ff2b5dc071e54b21e5a1e51aa2cee1676f1c986f/cryptography-46.0.3-cp314-cp314t-manylinux_2_34_aarch64.whl", hash = "sha256:23b1a8f26e43f47ceb6d6a43115f33a5a37d57df4ea0ca295b780ae8546e8044", size = 4292415, upload-time = "2025-10-15T23:17:31.686Z" },
 383      { url = "https://files.pythonhosted.org/packages/42/f9/2f8fefdb1aee8a8e3256a0568cffc4e6d517b256a2fe97a029b3f1b9fe7e/cryptography-46.0.3-cp314-cp314t-manylinux_2_34_ppc64le.whl", hash = "sha256:b419ae593c86b87014b9be7396b385491ad7f320bde96826d0dd174459e54665", size = 4931457, upload-time = "2025-10-15T23:17:33.478Z" },
 384      { url = "https://files.pythonhosted.org/packages/79/30/9b54127a9a778ccd6d27c3da7563e9f2d341826075ceab89ae3b41bf5be2/cryptography-46.0.3-cp314-cp314t-manylinux_2_34_x86_64.whl", hash = "sha256:50fc3343ac490c6b08c0cf0d704e881d0d660be923fd3076db3e932007e726e3", size = 4466074, upload-time = "2025-10-15T23:17:35.158Z" },
 385      { url = "https://files.pythonhosted.org/packages/ac/68/b4f4a10928e26c941b1b6a179143af9f4d27d88fe84a6a3c53592d2e76bf/cryptography-46.0.3-cp314-cp314t-musllinux_1_2_aarch64.whl", hash = "sha256:22d7e97932f511d6b0b04f2bfd818d73dcd5928db509460aaf48384778eb6d20", size = 4420569, upload-time = "2025-10-15T23:17:37.188Z" },
 386      { url = "https://files.pythonhosted.org/packages/a3/49/3746dab4c0d1979888f125226357d3262a6dd40e114ac29e3d2abdf1ec55/cryptography-46.0.3-cp314-cp314t-musllinux_1_2_x86_64.whl", hash = "sha256:d55f3dffadd674514ad19451161118fd010988540cee43d8bc20675e775925de", size = 4681941, upload-time = "2025-10-15T23:17:39.236Z" },
 387      { url = "https://files.pythonhosted.org/packages/fd/30/27654c1dbaf7e4a3531fa1fc77986d04aefa4d6d78259a62c9dc13d7ad36/cryptography-46.0.3-cp314-cp314t-win32.whl", hash = "sha256:8a6e050cb6164d3f830453754094c086ff2d0b2f3a897a1d9820f6139a1f0914", size = 3022339, upload-time = "2025-10-15T23:17:40.888Z" },
 388      { url = "https://files.pythonhosted.org/packages/f6/30/640f34ccd4d2a1bc88367b54b926b781b5a018d65f404d409aba76a84b1c/cryptography-46.0.3-cp314-cp314t-win_amd64.whl", hash = "sha256:760f83faa07f8b64e9c33fc963d790a2edb24efb479e3520c14a45741cd9b2db", size = 3494315, upload-time = "2025-10-15T23:17:42.769Z" },
 389      { url = "https://files.pythonhosted.org/packages/ba/8b/88cc7e3bd0a8e7b861f26981f7b820e1f46aa9d26cc482d0feba0ecb4919/cryptography-46.0.3-cp314-cp314t-win_arm64.whl", hash = "sha256:516ea134e703e9fe26bcd1277a4b59ad30586ea90c365a87781d7887a646fe21", size = 2919331, upload-time = "2025-10-15T23:17:44.468Z" },
 390      { url = "https://files.pythonhosted.org/packages/fd/23/45fe7f376a7df8daf6da3556603b36f53475a99ce4faacb6ba2cf3d82021/cryptography-46.0.3-cp38-abi3-macosx_10_9_universal2.whl", hash = "sha256:cb3d760a6117f621261d662bccc8ef5bc32ca673e037c83fbe565324f5c46936", size = 7218248, upload-time = "2025-10-15T23:17:46.294Z" },
 391      { url = "https://files.pythonhosted.org/packages/27/32/b68d27471372737054cbd34c84981f9edbc24fe67ca225d389799614e27f/cryptography-46.0.3-cp38-abi3-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:4b7387121ac7d15e550f5cb4a43aef2559ed759c35df7336c402bb8275ac9683", size = 4294089, upload-time = "2025-10-15T23:17:48.269Z" },
 392      { url = "https://files.pythonhosted.org/packages/26/42/fa8389d4478368743e24e61eea78846a0006caffaf72ea24a15159215a14/cryptography-46.0.3-cp38-abi3-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:15ab9b093e8f09daab0f2159bb7e47532596075139dd74365da52ecc9cb46c5d", size = 4440029, upload-time = "2025-10-15T23:17:49.837Z" },
 393      { url = "https://files.pythonhosted.org/packages/5f/eb/f483db0ec5ac040824f269e93dd2bd8a21ecd1027e77ad7bdf6914f2fd80/cryptography-46.0.3-cp38-abi3-manylinux_2_28_aarch64.whl", hash = "sha256:46acf53b40ea38f9c6c229599a4a13f0d46a6c3fa9ef19fc1a124d62e338dfa0", size = 4297222, upload-time = "2025-10-15T23:17:51.357Z" },
 394      { url = "https://files.pythonhosted.org/packages/fd/cf/da9502c4e1912cb1da3807ea3618a6829bee8207456fbbeebc361ec38ba3/cryptography-46.0.3-cp38-abi3-manylinux_2_28_armv7l.manylinux_2_31_armv7l.whl", hash = "sha256:10ca84c4668d066a9878890047f03546f3ae0a6b8b39b697457b7757aaf18dbc", size = 4012280, upload-time = "2025-10-15T23:17:52.964Z" },
 395      { url = "https://files.pythonhosted.org/packages/6b/8f/9adb86b93330e0df8b3dcf03eae67c33ba89958fc2e03862ef1ac2b42465/cryptography-46.0.3-cp38-abi3-manylinux_2_28_ppc64le.whl", hash = "sha256:36e627112085bb3b81b19fed209c05ce2a52ee8b15d161b7c643a7d5a88491f3", size = 4978958, upload-time = "2025-10-15T23:17:54.965Z" },
 396      { url = "https://files.pythonhosted.org/packages/d1/a0/5fa77988289c34bdb9f913f5606ecc9ada1adb5ae870bd0d1054a7021cc4/cryptography-46.0.3-cp38-abi3-manylinux_2_28_x86_64.whl", hash = "sha256:1000713389b75c449a6e979ffc7dcc8ac90b437048766cef052d4d30b8220971", size = 4473714, upload-time = "2025-10-15T23:17:56.754Z" },
 397      { url = "https://files.pythonhosted.org/packages/14/e5/fc82d72a58d41c393697aa18c9abe5ae1214ff6f2a5c18ac470f92777895/cryptography-46.0.3-cp38-abi3-manylinux_2_34_aarch64.whl", hash = "sha256:b02cf04496f6576afffef5ddd04a0cb7d49cf6be16a9059d793a30b035f6b6ac", size = 4296970, upload-time = "2025-10-15T23:17:58.588Z" },
 398      { url = "https://files.pythonhosted.org/packages/78/06/5663ed35438d0b09056973994f1aec467492b33bd31da36e468b01ec1097/cryptography-46.0.3-cp38-abi3-manylinux_2_34_ppc64le.whl", hash = "sha256:71e842ec9bc7abf543b47cf86b9a743baa95f4677d22baa4c7d5c69e49e9bc04", size = 4940236, upload-time = "2025-10-15T23:18:00.897Z" },
 399      { url = "https://files.pythonhosted.org/packages/fc/59/873633f3f2dcd8a053b8dd1d38f783043b5fce589c0f6988bf55ef57e43e/cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl", hash = "sha256:402b58fc32614f00980b66d6e56a5b4118e6cb362ae8f3fda141ba4689bd4506", size = 4472642, upload-time = "2025-10-15T23:18:02.749Z" },
 400      { url = "https://files.pythonhosted.org/packages/3d/39/8e71f3930e40f6877737d6f69248cf74d4e34b886a3967d32f919cc50d3b/cryptography-46.0.3-cp38-abi3-musllinux_1_2_aarch64.whl", hash = "sha256:ef639cb3372f69ec44915fafcd6698b6cc78fbe0c2ea41be867f6ed612811963", size = 4423126, upload-time = "2025-10-15T23:18:04.85Z" },
 401      { url = "https://files.pythonhosted.org/packages/cd/c7/f65027c2810e14c3e7268353b1681932b87e5a48e65505d8cc17c99e36ae/cryptography-46.0.3-cp38-abi3-musllinux_1_2_x86_64.whl", hash = "sha256:3b51b8ca4f1c6453d8829e1eb7299499ca7f313900dd4d89a24b8b87c0a780d4", size = 4686573, upload-time = "2025-10-15T23:18:06.908Z" },
 402      { url = "https://files.pythonhosted.org/packages/0a/6e/1c8331ddf91ca4730ab3086a0f1be19c65510a33b5a441cb334e7a2d2560/cryptography-46.0.3-cp38-abi3-win32.whl", hash = "sha256:6276eb85ef938dc035d59b87c8a7dc559a232f954962520137529d77b18ff1df", size = 3036695, upload-time = "2025-10-15T23:18:08.672Z" },
 403      { url = "https://files.pythonhosted.org/packages/90/45/b0d691df20633eff80955a0fc7695ff9051ffce8b69741444bd9ed7bd0db/cryptography-46.0.3-cp38-abi3-win_amd64.whl", hash = "sha256:416260257577718c05135c55958b674000baef9a1c7d9e8f306ec60d71db850f", size = 3501720, upload-time = "2025-10-15T23:18:10.632Z" },
 404      { url = "https://files.pythonhosted.org/packages/e8/cb/2da4cc83f5edb9c3257d09e1e7ab7b23f049c7962cae8d842bbef0a9cec9/cryptography-46.0.3-cp38-abi3-win_arm64.whl", hash = "sha256:d89c3468de4cdc4f08a57e214384d0471911a3830fcdaf7a8cc587e42a866372", size = 2918740, upload-time = "2025-10-15T23:18:12.277Z" },
 405      { url = "https://files.pythonhosted.org/packages/d9/cd/1a8633802d766a0fa46f382a77e096d7e209e0817892929655fe0586ae32/cryptography-46.0.3-pp310-pypy310_pp73-macosx_10_9_x86_64.whl", hash = "sha256:a23582810fedb8c0bc47524558fb6c56aac3fc252cb306072fd2815da2a47c32", size = 3689163, upload-time = "2025-10-15T23:18:13.821Z" },
 406      { url = "https://files.pythonhosted.org/packages/4c/59/6b26512964ace6480c3e54681a9859c974172fb141c38df11eadd8416947/cryptography-46.0.3-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:e7aec276d68421f9574040c26e2a7c3771060bc0cff408bae1dcb19d3ab1e63c", size = 3429474, upload-time = "2025-10-15T23:18:15.477Z" },
 407      { url = "https://files.pythonhosted.org/packages/06/8a/e60e46adab4362a682cf142c7dcb5bf79b782ab2199b0dcb81f55970807f/cryptography-46.0.3-pp311-pypy311_pp73-macosx_10_9_x86_64.whl", hash = "sha256:7ce938a99998ed3c8aa7e7272dca1a610401ede816d36d0693907d863b10d9ea", size = 3698132, upload-time = "2025-10-15T23:18:17.056Z" },
 408      { url = "https://files.pythonhosted.org/packages/da/38/f59940ec4ee91e93d3311f7532671a5cef5570eb04a144bf203b58552d11/cryptography-46.0.3-pp311-pypy311_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:191bb60a7be5e6f54e30ba16fdfae78ad3a342a0599eb4193ba88e3f3d6e185b", size = 4243992, upload-time = "2025-10-15T23:18:18.695Z" },
 409      { url = "https://files.pythonhosted.org/packages/b0/0c/35b3d92ddebfdfda76bb485738306545817253d0a3ded0bfe80ef8e67aa5/cryptography-46.0.3-pp311-pypy311_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:c70cc23f12726be8f8bc72e41d5065d77e4515efae3690326764ea1b07845cfb", size = 4409944, upload-time = "2025-10-15T23:18:20.597Z" },
 410      { url = "https://files.pythonhosted.org/packages/99/55/181022996c4063fc0e7666a47049a1ca705abb9c8a13830f074edb347495/cryptography-46.0.3-pp311-pypy311_pp73-manylinux_2_34_aarch64.whl", hash = "sha256:9394673a9f4de09e28b5356e7fff97d778f8abad85c9d5ac4a4b7e25a0de7717", size = 4242957, upload-time = "2025-10-15T23:18:22.18Z" },
 411      { url = "https://files.pythonhosted.org/packages/ba/af/72cd6ef29f9c5f731251acadaeb821559fe25f10852f44a63374c9ca08c1/cryptography-46.0.3-pp311-pypy311_pp73-manylinux_2_34_x86_64.whl", hash = "sha256:94cd0549accc38d1494e1f8de71eca837d0509d0d44bf11d158524b0e12cebf9", size = 4409447, upload-time = "2025-10-15T23:18:24.209Z" },
 412      { url = "https://files.pythonhosted.org/packages/0d/c3/e90f4a4feae6410f914f8ebac129b9ae7a8c92eb60a638012dde42030a9d/cryptography-46.0.3-pp311-pypy311_pp73-win_amd64.whl", hash = "sha256:6b5063083824e5509fdba180721d55909ffacccc8adbec85268b48439423d78c", size = 3438528, upload-time = "2025-10-15T23:18:26.227Z" },
 413  ]
 414  
 415  [[package]]
 416  name = "exceptiongroup"
 417  version = "1.3.0"
 418  source = { registry = "https://pypi.org/simple" }
 419  dependencies = [
 420      { name = "typing-extensions", version = "4.13.2", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 421      { name = "typing-extensions", version = "4.15.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.9' and python_full_version < '3.13'" },
 422  ]
 423  sdist = { url = "https://files.pythonhosted.org/packages/0b/9f/a65090624ecf468cdca03533906e7c69ed7588582240cfe7cc9e770b50eb/exceptiongroup-1.3.0.tar.gz", hash = "sha256:b241f5885f560bc56a59ee63ca4c6a8bfa46ae4ad651af316d4e81817bb9fd88", size = 29749, upload-time = "2025-05-10T17:42:51.123Z" }
 424  wheels = [
 425      { url = "https://files.pythonhosted.org/packages/36/f4/c6e662dade71f56cd2f3735141b265c3c79293c109549c1e6933b0651ffc/exceptiongroup-1.3.0-py3-none-any.whl", hash = "sha256:4d111e6e0c13d0644cad6ddaa7ed0261a0b36971f6d23e7ec9b4b9097da78a10", size = 16674, upload-time = "2025-05-10T17:42:49.33Z" },
 426  ]
 427  
 428  [[package]]
 429  name = "files-to-prompt"
 430  version = "0.6"
 431  source = { virtual = "." }
 432  dependencies = [
 433      { name = "click", version = "8.1.8", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.10'" },
 434      { name = "click", version = "8.3.1", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.10'" },
 435      { name = "pandoc" },
 436      { name = "pdfplumber", version = "0.11.5", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 437      { name = "pdfplumber", version = "0.11.8", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.9'" },
 438  ]
 439  
 440  [package.optional-dependencies]
 441  test = [
 442      { name = "pytest", version = "8.3.5", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 443      { name = "pytest", version = "8.4.2", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version == '3.9.*'" },
 444      { name = "pytest", version = "9.0.1", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.10'" },
 445  ]
 446  
 447  [package.metadata]
 448  requires-dist = [
 449      { name = "click" },
 450      { name = "pandoc", specifier = ">=2.4" },
 451      { name = "pdfplumber" },
 452      { name = "pytest", marker = "extra == 'test'" },
 453  ]
 454  provides-extras = ["test"]
 455  
 456  [[package]]
 457  name = "importlib-resources"
 458  version = "6.4.5"
 459  source = { registry = "https://pypi.org/simple" }
 460  dependencies = [
 461      { name = "zipp", marker = "python_full_version < '3.9'" },
 462  ]
 463  sdist = { url = "https://files.pythonhosted.org/packages/98/be/f3e8c6081b684f176b761e6a2fef02a0be939740ed6f54109a2951d806f3/importlib_resources-6.4.5.tar.gz", hash = "sha256:980862a1d16c9e147a59603677fa2aa5fd82b87f223b6cb870695bcfce830065", size = 43372, upload-time = "2024-09-09T17:03:14.677Z" }
 464  wheels = [
 465      { url = "https://files.pythonhosted.org/packages/e1/6a/4604f9ae2fa62ef47b9de2fa5ad599589d28c9fd1d335f32759813dfa91e/importlib_resources-6.4.5-py3-none-any.whl", hash = "sha256:ac29d5f956f01d5e4bb63102a5a19957f1b9175e45649977264a1416783bb717", size = 36115, upload-time = "2024-09-09T17:03:13.39Z" },
 466  ]
 467  
 468  [[package]]
 469  name = "iniconfig"
 470  version = "2.1.0"
 471  source = { registry = "https://pypi.org/simple" }
 472  resolution-markers = [
 473      "python_full_version == '3.9.*'",
 474      "python_full_version < '3.9'",
 475  ]
 476  sdist = { url = "https://files.pythonhosted.org/packages/f2/97/ebf4da567aa6827c909642694d71c9fcf53e5b504f2d96afea02718862f3/iniconfig-2.1.0.tar.gz", hash = "sha256:3abbd2e30b36733fee78f9c7f7308f2d0050e88f0087fd25c2645f63c773e1c7", size = 4793, upload-time = "2025-03-19T20:09:59.721Z" }
 477  wheels = [
 478      { url = "https://files.pythonhosted.org/packages/2c/e1/e6716421ea10d38022b952c159d5161ca1193197fb744506875fbb87ea7b/iniconfig-2.1.0-py3-none-any.whl", hash = "sha256:9deba5723312380e77435581c6bf4935c94cbfab9b1ed33ef8d238ea168eb760", size = 6050, upload-time = "2025-03-19T20:10:01.071Z" },
 479  ]
 480  
 481  [[package]]
 482  name = "iniconfig"
 483  version = "2.3.0"
 484  source = { registry = "https://pypi.org/simple" }
 485  resolution-markers = [
 486      "python_full_version >= '3.10'",
 487  ]
 488  sdist = { url = "https://files.pythonhosted.org/packages/72/34/14ca021ce8e5dfedc35312d08ba8bf51fdd999c576889fc2c24cb97f4f10/iniconfig-2.3.0.tar.gz", hash = "sha256:c76315c77db068650d49c5b56314774a7804df16fee4402c1f19d6d15d8c4730", size = 20503, upload-time = "2025-10-18T21:55:43.219Z" }
 489  wheels = [
 490      { url = "https://files.pythonhosted.org/packages/cb/b1/3846dd7f199d53cb17f49cba7e651e9ce294d8497c8c150530ed11865bb8/iniconfig-2.3.0-py3-none-any.whl", hash = "sha256:f631c04d2c48c52b84d0d0549c99ff3859c98df65b3101406327ecc7d53fbf12", size = 7484, upload-time = "2025-10-18T21:55:41.639Z" },
 491  ]
 492  
 493  [[package]]
 494  name = "packaging"
 495  version = "25.0"
 496  source = { registry = "https://pypi.org/simple" }
 497  sdist = { url = "https://files.pythonhosted.org/packages/a1/d4/1fc4078c65507b51b96ca8f8c3ba19e6a61c8253c72794544580a7b6c24d/packaging-25.0.tar.gz", hash = "sha256:d443872c98d677bf60f6a1f2f8c1cb748e8fe762d2bf9d3148b5599295b0fc4f", size = 165727, upload-time = "2025-04-19T11:48:59.673Z" }
 498  wheels = [
 499      { url = "https://files.pythonhosted.org/packages/20/12/38679034af332785aac8774540895e234f4d07f7545804097de4b666afd8/packaging-25.0-py3-none-any.whl", hash = "sha256:29572ef2b1f17581046b3a2227d5c611fb25ec70ca1ba8554b24b0e69331a484", size = 66469, upload-time = "2025-04-19T11:48:57.875Z" },
 500  ]
 501  
 502  [[package]]
 503  name = "pandoc"
 504  version = "2.4"
 505  source = { registry = "https://pypi.org/simple" }
 506  dependencies = [
 507      { name = "plumbum", version = "1.9.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 508      { name = "plumbum", version = "1.10.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.9'" },
 509      { name = "ply" },
 510  ]
 511  sdist = { url = "https://files.pythonhosted.org/packages/10/9a/e3186e760c57ee5f1c27ea5cea577a0ff9abfca51eefcb4d9a4cd39aff2e/pandoc-2.4.tar.gz", hash = "sha256:ecd1f8cbb7f4180c6b5db4a17a7c1a74df519995f5f186ef81ce72a9cbd0dd9a", size = 34635, upload-time = "2024-08-07T14:33:58.016Z" }
 512  
 513  [[package]]
 514  name = "pdfminer-six"
 515  version = "20231228"
 516  source = { registry = "https://pypi.org/simple" }
 517  resolution-markers = [
 518      "python_full_version < '3.9'",
 519  ]
 520  dependencies = [
 521      { name = "charset-normalizer", marker = "python_full_version < '3.9'" },
 522      { name = "cryptography", marker = "python_full_version < '3.9'" },
 523  ]
 524  sdist = { url = "https://files.pythonhosted.org/packages/31/b1/a43e3bd872ded4deea4f8efc7aff1703fca8c5455d0c06e20506a06a44ff/pdfminer.six-20231228.tar.gz", hash = "sha256:6004da3ad1a7a4d45930cb950393df89b068e73be365a6ff64a838d37bcb08c4", size = 7362505, upload-time = "2023-12-28T21:25:32.863Z" }
 525  wheels = [
 526      { url = "https://files.pythonhosted.org/packages/eb/9c/e46fe7502b32d7db6af6e36a9105abb93301fa1ec475b5ddcba8b35ae23a/pdfminer.six-20231228-py3-none-any.whl", hash = "sha256:e8d3c3310e6fbc1fe414090123ab01351634b4ecb021232206c4c9a8ca3e3b8f", size = 5614515, upload-time = "2023-12-28T21:25:30.329Z" },
 527  ]
 528  
 529  [[package]]
 530  name = "pdfminer-six"
 531  version = "20251107"
 532  source = { registry = "https://pypi.org/simple" }
 533  resolution-markers = [
 534      "python_full_version >= '3.10'",
 535      "python_full_version == '3.9.*'",
 536  ]
 537  dependencies = [
 538      { name = "charset-normalizer", marker = "python_full_version >= '3.9'" },
 539      { name = "cryptography", marker = "python_full_version >= '3.9'" },
 540  ]
 541  sdist = { url = "https://files.pythonhosted.org/packages/1d/50/5315f381a25dc80a8d2ea7c62d9a28c0137f10ccc263623a0db8b49fcced/pdfminer_six-20251107.tar.gz", hash = "sha256:5fb0c553799c591777f22c0c72b77fc2522d7d10c70654e25f4c5f1fd996e008", size = 7387104, upload-time = "2025-11-07T20:01:10.286Z" }
 542  wheels = [
 543      { url = "https://files.pythonhosted.org/packages/64/29/d1d9f6b900191288b77613ddefb73ed35b48fb35e44aaf8b01b0422b759d/pdfminer_six-20251107-py3-none-any.whl", hash = "sha256:c09df33e4cbe6b26b2a79248a4ffcccafaa5c5d39c9fff0e6e81567f165b5401", size = 5620299, upload-time = "2025-11-07T20:01:08.722Z" },
 544  ]
 545  
 546  [[package]]
 547  name = "pdfplumber"
 548  version = "0.11.5"
 549  source = { registry = "https://pypi.org/simple" }
 550  resolution-markers = [
 551      "python_full_version < '3.9'",
 552  ]
 553  dependencies = [
 554      { name = "pdfminer-six", version = "20231228", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 555      { name = "pillow", version = "10.4.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
 556      { name = "pypdfium2", marker = "python_full_version < '3.9'" },
 557  ]
 558  sdist = { url = "https://files.pythonhosted.org/packages/28/55/33d265185b4e7e6ac81e64478750ea26310055b1b5b278851b981a1c57e5/pdfplumber-0.11.5.tar.gz", hash = "sha256:dadd81b62a0b23e078cdd89de26e013850d4daf5690fcf46dec396b07e6737d6", size = 114626, upload-time = "2025-01-01T15:39:14.709Z" }
 559  wheels = [
 560      { url = "https://files.pythonhosted.org/packages/18/fe/eebf169301bd1c1c69d639e81ba226d333d35c5ad105b7cd3cfc40a44862/pdfplumber-0.11.5-py3-none-any.whl", hash = "sha256:a6e0921a57e0ef7356001a0fd811250b0e37a0b42630a922ee48f55cdd534070", size = 59515, upload-time = "2025-01-01T15:39:12.206Z" },
 561  ]
 562  
 563  [[package]]
 564  name = "pdfplumber"
 565  version = "0.11.8"
 566  source = { registry = "https://pypi.org/simple" }
 567  resolution-markers = [
 568      "python_full_version >= '3.10'",
 569      "python_full_version == '3.9.*'",
 570  ]
 571  dependencies = [
 572      { name = "pdfminer-six", version = "20251107", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.9'" },
 573      { name = "pillow", version = "11.3.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version == '3.9.*'" },
 574      { name = "pillow", version = "12.0.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.10'" },
 575      { name = "pypdfium2", marker = "python_full_version >= '3.9'" },
 576  ]
 577  sdist = { url = "https://files.pythonhosted.org/packages/09/d8/cb9fda4261ce389656bec0bb0bdde905df109ad97f7ae387747ded070e8c/pdfplumber-0.11.8.tar.gz", hash = "sha256:db29b04bc8bb62f39dd444533bcf2e0ba33584bd24f5a54644f3ba30f4f22d31", size = 102724, upload-time = "2025-11-08T20:52:01.955Z" }
 578  wheels = [
 579      { url = "https://files.pythonhosted.org/packages/12/28/3958ed81a9be317610ab73df32f1968076751d651c84dff1bcb45b7c6c0e/pdfplumber-0.11.8-py3-none-any.whl", hash = "sha256:7dda117b8ed21bca9c8e7d7808fee2439f93c8bd6ea45989bfb1aead6dc3cad3", size = 60043, upload-time = "2025-11-08T20:52:00.652Z" },
 580  ]
 581  
 582  [[package]]
 583  name = "pillow"
 584  version = "10.4.0"
 585  source = { registry = "https://pypi.org/simple" }
 586  resolution-markers = [
 587      "python_full_version < '3.9'",
 588  ]
 589  sdist = { url = "https://files.pythonhosted.org/packages/cd/74/ad3d526f3bf7b6d3f408b73fde271ec69dfac8b81341a318ce825f2b3812/pillow-10.4.0.tar.gz", hash = "sha256:166c1cd4d24309b30d61f79f4a9114b7b2313d7450912277855ff5dfd7cd4a06", size = 46555059, upload-time = "2024-07-01T09:48:43.583Z" }
 590  wheels = [
 591      { url = "https://files.pythonhosted.org/packages/0e/69/a31cccd538ca0b5272be2a38347f8839b97a14be104ea08b0db92f749c74/pillow-10.4.0-cp310-cp310-macosx_10_10_x86_64.whl", hash = "sha256:4d9667937cfa347525b319ae34375c37b9ee6b525440f3ef48542fcf66f2731e", size = 3509271, upload-time = "2024-07-01T09:45:22.07Z" },
 592      { url = "https://files.pythonhosted.org/packages/9a/9e/4143b907be8ea0bce215f2ae4f7480027473f8b61fcedfda9d851082a5d2/pillow-10.4.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:543f3dc61c18dafb755773efc89aae60d06b6596a63914107f75459cf984164d", size = 3375658, upload-time = "2024-07-01T09:45:25.292Z" },
 593      { url = "https://files.pythonhosted.org/packages/8a/25/1fc45761955f9359b1169aa75e241551e74ac01a09f487adaaf4c3472d11/pillow-10.4.0-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:7928ecbf1ece13956b95d9cbcfc77137652b02763ba384d9ab508099a2eca856", size = 4332075, upload-time = "2024-07-01T09:45:27.94Z" },
 594      { url = "https://files.pythonhosted.org/packages/5e/dd/425b95d0151e1d6c951f45051112394f130df3da67363b6bc75dc4c27aba/pillow-10.4.0-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:e4d49b85c4348ea0b31ea63bc75a9f3857869174e2bf17e7aba02945cd218e6f", size = 4444808, upload-time = "2024-07-01T09:45:30.305Z" },
 595      { url = "https://files.pythonhosted.org/packages/b1/84/9a15cc5726cbbfe7f9f90bfb11f5d028586595907cd093815ca6644932e3/pillow-10.4.0-cp310-cp310-manylinux_2_28_aarch64.whl", hash = "sha256:6c762a5b0997f5659a5ef2266abc1d8851ad7749ad9a6a5506eb23d314e4f46b", size = 4356290, upload-time = "2024-07-01T09:45:32.868Z" },
 596      { url = "https://files.pythonhosted.org/packages/b5/5b/6651c288b08df3b8c1e2f8c1152201e0b25d240e22ddade0f1e242fc9fa0/pillow-10.4.0-cp310-cp310-manylinux_2_28_x86_64.whl", hash = "sha256:a985e028fc183bf12a77a8bbf36318db4238a3ded7fa9df1b9a133f1cb79f8fc", size = 4525163, upload-time = "2024-07-01T09:45:35.279Z" },
 597      { url = "https://files.pythonhosted.org/packages/07/8b/34854bf11a83c248505c8cb0fcf8d3d0b459a2246c8809b967963b6b12ae/pillow-10.4.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:812f7342b0eee081eaec84d91423d1b4650bb9828eb53d8511bcef8ce5aecf1e", size = 4463100, upload-time = "2024-07-01T09:45:37.74Z" },
 598      { url = "https://files.pythonhosted.org/packages/78/63/0632aee4e82476d9cbe5200c0cdf9ba41ee04ed77887432845264d81116d/pillow-10.4.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:ac1452d2fbe4978c2eec89fb5a23b8387aba707ac72810d9490118817d9c0b46", size = 4592880, upload-time = "2024-07-01T09:45:39.89Z" },
 599      { url = "https://files.pythonhosted.org/packages/df/56/b8663d7520671b4398b9d97e1ed9f583d4afcbefbda3c6188325e8c297bd/pillow-10.4.0-cp310-cp310-win32.whl", hash = "sha256:bcd5e41a859bf2e84fdc42f4edb7d9aba0a13d29a2abadccafad99de3feff984", size = 2235218, upload-time = "2024-07-01T09:45:42.771Z" },
 600      { url = "https://files.pythonhosted.org/packages/f4/72/0203e94a91ddb4a9d5238434ae6c1ca10e610e8487036132ea9bf806ca2a/pillow-10.4.0-cp310-cp310-win_amd64.whl", hash = "sha256:ecd85a8d3e79cd7158dec1c9e5808e821feea088e2f69a974db5edf84dc53141", size = 2554487, upload-time = "2024-07-01T09:45:45.176Z" },
 601      { url = "https://files.pythonhosted.org/packages/bd/52/7e7e93d7a6e4290543f17dc6f7d3af4bd0b3dd9926e2e8a35ac2282bc5f4/pillow-10.4.0-cp310-cp310-win_arm64.whl", hash = "sha256:ff337c552345e95702c5fde3158acb0625111017d0e5f24bf3acdb9cc16b90d1", size = 2243219, upload-time = "2024-07-01T09:45:47.274Z" },
 602      { url = "https://files.pythonhosted.org/packages/a7/62/c9449f9c3043c37f73e7487ec4ef0c03eb9c9afc91a92b977a67b3c0bbc5/pillow-10.4.0-cp311-cp311-macosx_10_10_x86_64.whl", hash = "sha256:0a9ec697746f268507404647e531e92889890a087e03681a3606d9b920fbee3c", size = 3509265, upload-time = "2024-07-01T09:45:49.812Z" },
 603      { url = "https://files.pythonhosted.org/packages/f4/5f/491dafc7bbf5a3cc1845dc0430872e8096eb9e2b6f8161509d124594ec2d/pillow-10.4.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:dfe91cb65544a1321e631e696759491ae04a2ea11d36715eca01ce07284738be", size = 3375655, upload-time = "2024-07-01T09:45:52.462Z" },
 604      { url = "https://files.pythonhosted.org/packages/73/d5/c4011a76f4207a3c151134cd22a1415741e42fa5ddecec7c0182887deb3d/pillow-10.4.0-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:5dc6761a6efc781e6a1544206f22c80c3af4c8cf461206d46a1e6006e4429ff3", size = 4340304, upload-time = "2024-07-01T09:45:55.006Z" },
 605      { url = "https://files.pythonhosted.org/packages/ac/10/c67e20445a707f7a610699bba4fe050583b688d8cd2d202572b257f46600/pillow-10.4.0-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:5e84b6cc6a4a3d76c153a6b19270b3526a5a8ed6b09501d3af891daa2a9de7d6", size = 4452804, upload-time = "2024-07-01T09:45:58.437Z" },
 606      { url = "https://files.pythonhosted.org/packages/a9/83/6523837906d1da2b269dee787e31df3b0acb12e3d08f024965a3e7f64665/pillow-10.4.0-cp311-cp311-manylinux_2_28_aarch64.whl", hash = "sha256:bbc527b519bd3aa9d7f429d152fea69f9ad37c95f0b02aebddff592688998abe", size = 4365126, upload-time = "2024-07-01T09:46:00.713Z" },
 607      { url = "https://files.pythonhosted.org/packages/ba/e5/8c68ff608a4203085158cff5cc2a3c534ec384536d9438c405ed6370d080/pillow-10.4.0-cp311-cp311-manylinux_2_28_x86_64.whl", hash = "sha256:76a911dfe51a36041f2e756b00f96ed84677cdeb75d25c767f296c1c1eda1319", size = 4533541, upload-time = "2024-07-01T09:46:03.235Z" },
 608      { url = "https://files.pythonhosted.org/packages/f4/7c/01b8dbdca5bc6785573f4cee96e2358b0918b7b2c7b60d8b6f3abf87a070/pillow-10.4.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:59291fb29317122398786c2d44427bbd1a6d7ff54017075b22be9d21aa59bd8d", size = 4471616, upload-time = "2024-07-01T09:46:05.356Z" },
 609      { url = "https://files.pythonhosted.org/packages/c8/57/2899b82394a35a0fbfd352e290945440e3b3785655a03365c0ca8279f351/pillow-10.4.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:416d3a5d0e8cfe4f27f574362435bc9bae57f679a7158e0096ad2beb427b8696", size = 4600802, upload-time = "2024-07-01T09:46:08.145Z" },
 610      { url = "https://files.pythonhosted.org/packages/4d/d7/a44f193d4c26e58ee5d2d9db3d4854b2cfb5b5e08d360a5e03fe987c0086/pillow-10.4.0-cp311-cp311-win32.whl", hash = "sha256:7086cc1d5eebb91ad24ded9f58bec6c688e9f0ed7eb3dbbf1e4800280a896496", size = 2235213, upload-time = "2024-07-01T09:46:10.211Z" },
 611      { url = "https://files.pythonhosted.org/packages/c1/d0/5866318eec2b801cdb8c82abf190c8343d8a1cd8bf5a0c17444a6f268291/pillow-10.4.0-cp311-cp311-win_amd64.whl", hash = "sha256:cbed61494057c0f83b83eb3a310f0bf774b09513307c434d4366ed64f4128a91", size = 2554498, upload-time = "2024-07-01T09:46:12.685Z" },
 612      { url = "https://files.pythonhosted.org/packages/d4/c8/310ac16ac2b97e902d9eb438688de0d961660a87703ad1561fd3dfbd2aa0/pillow-10.4.0-cp311-cp311-win_arm64.whl", hash = "sha256:f5f0c3e969c8f12dd2bb7e0b15d5c468b51e5017e01e2e867335c81903046a22", size = 2243219, upload-time = "2024-07-01T09:46:14.83Z" },
 613      { url = "https://files.pythonhosted.org/packages/05/cb/0353013dc30c02a8be34eb91d25e4e4cf594b59e5a55ea1128fde1e5f8ea/pillow-10.4.0-cp312-cp312-macosx_10_10_x86_64.whl", hash = "sha256:673655af3eadf4df6b5457033f086e90299fdd7a47983a13827acf7459c15d94", size = 3509350, upload-time = "2024-07-01T09:46:17.177Z" },
 614      { url = "https://files.pythonhosted.org/packages/e7/cf/5c558a0f247e0bf9cec92bff9b46ae6474dd736f6d906315e60e4075f737/pillow-10.4.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:866b6942a92f56300012f5fbac71f2d610312ee65e22f1aa2609e491284e5597", size = 3374980, upload-time = "2024-07-01T09:46:19.169Z" },
 615      { url = "https://files.pythonhosted.org/packages/84/48/6e394b86369a4eb68b8a1382c78dc092245af517385c086c5094e3b34428/pillow-10.4.0-cp312-cp312-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:29dbdc4207642ea6aad70fbde1a9338753d33fb23ed6956e706936706f52dd80", size = 4343799, upload-time = "2024-07-01T09:46:21.883Z" },
 616      { url = "https://files.pythonhosted.org/packages/3b/f3/a8c6c11fa84b59b9df0cd5694492da8c039a24cd159f0f6918690105c3be/pillow-10.4.0-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:bf2342ac639c4cf38799a44950bbc2dfcb685f052b9e262f446482afaf4bffca", size = 4459973, upload-time = "2024-07-01T09:46:24.321Z" },
 617      { url = "https://files.pythonhosted.org/packages/7d/1b/c14b4197b80150fb64453585247e6fb2e1d93761fa0fa9cf63b102fde822/pillow-10.4.0-cp312-cp312-manylinux_2_28_aarch64.whl", hash = "sha256:f5b92f4d70791b4a67157321c4e8225d60b119c5cc9aee8ecf153aace4aad4ef", size = 4370054, upload-time = "2024-07-01T09:46:26.825Z" },
 618      { url = "https://files.pythonhosted.org/packages/55/77/40daddf677897a923d5d33329acd52a2144d54a9644f2a5422c028c6bf2d/pillow-10.4.0-cp312-cp312-manylinux_2_28_x86_64.whl", hash = "sha256:86dcb5a1eb778d8b25659d5e4341269e8590ad6b4e8b44d9f4b07f8d136c414a", size = 4539484, upload-time = "2024-07-01T09:46:29.355Z" },
 619      { url = "https://files.pythonhosted.org/packages/40/54/90de3e4256b1207300fb2b1d7168dd912a2fb4b2401e439ba23c2b2cabde/pillow-10.4.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:780c072c2e11c9b2c7ca37f9a2ee8ba66f44367ac3e5c7832afcfe5104fd6d1b", size = 4477375, upload-time = "2024-07-01T09:46:31.756Z" },
 620      { url = "https://files.pythonhosted.org/packages/13/24/1bfba52f44193860918ff7c93d03d95e3f8748ca1de3ceaf11157a14cf16/pillow-10.4.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:37fb69d905be665f68f28a8bba3c6d3223c8efe1edf14cc4cfa06c241f8c81d9", size = 4608773, upload-time = "2024-07-01T09:46:33.73Z" },
 621      { url = "https://files.pythonhosted.org/packages/55/04/5e6de6e6120451ec0c24516c41dbaf80cce1b6451f96561235ef2429da2e/pillow-10.4.0-cp312-cp312-win32.whl", hash = "sha256:7dfecdbad5c301d7b5bde160150b4db4c659cee2b69589705b6f8a0c509d9f42", size = 2235690, upload-time = "2024-07-01T09:46:36.587Z" },
 622      { url = "https://files.pythonhosted.org/packages/74/0a/d4ce3c44bca8635bd29a2eab5aa181b654a734a29b263ca8efe013beea98/pillow-10.4.0-cp312-cp312-win_amd64.whl", hash = "sha256:1d846aea995ad352d4bdcc847535bd56e0fd88d36829d2c90be880ef1ee4668a", size = 2554951, upload-time = "2024-07-01T09:46:38.777Z" },
 623      { url = "https://files.pythonhosted.org/packages/b5/ca/184349ee40f2e92439be9b3502ae6cfc43ac4b50bc4fc6b3de7957563894/pillow-10.4.0-cp312-cp312-win_arm64.whl", hash = "sha256:e553cad5179a66ba15bb18b353a19020e73a7921296a7979c4a2b7f6a5cd57f9", size = 2243427, upload-time = "2024-07-01T09:46:43.15Z" },
 624      { url = "https://files.pythonhosted.org/packages/c3/00/706cebe7c2c12a6318aabe5d354836f54adff7156fd9e1bd6c89f4ba0e98/pillow-10.4.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:8bc1a764ed8c957a2e9cacf97c8b2b053b70307cf2996aafd70e91a082e70df3", size = 3525685, upload-time = "2024-07-01T09:46:45.194Z" },
 625      { url = "https://files.pythonhosted.org/packages/cf/76/f658cbfa49405e5ecbfb9ba42d07074ad9792031267e782d409fd8fe7c69/pillow-10.4.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:6209bb41dc692ddfee4942517c19ee81b86c864b626dbfca272ec0f7cff5d9fb", size = 3374883, upload-time = "2024-07-01T09:46:47.331Z" },
 626      { url = "https://files.pythonhosted.org/packages/46/2b/99c28c4379a85e65378211971c0b430d9c7234b1ec4d59b2668f6299e011/pillow-10.4.0-cp313-cp313-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:bee197b30783295d2eb680b311af15a20a8b24024a19c3a26431ff83eb8d1f70", size = 4339837, upload-time = "2024-07-01T09:46:49.647Z" },
 627      { url = "https://files.pythonhosted.org/packages/f1/74/b1ec314f624c0c43711fdf0d8076f82d9d802afd58f1d62c2a86878e8615/pillow-10.4.0-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:1ef61f5dd14c300786318482456481463b9d6b91ebe5ef12f405afbba77ed0be", size = 4455562, upload-time = "2024-07-01T09:46:51.811Z" },
 628      { url = "https://files.pythonhosted.org/packages/4a/2a/4b04157cb7b9c74372fa867096a1607e6fedad93a44deeff553ccd307868/pillow-10.4.0-cp313-cp313-manylinux_2_28_aarch64.whl", hash = "sha256:297e388da6e248c98bc4a02e018966af0c5f92dfacf5a5ca22fa01cb3179bca0", size = 4366761, upload-time = "2024-07-01T09:46:53.961Z" },
 629      { url = "https://files.pythonhosted.org/packages/ac/7b/8f1d815c1a6a268fe90481232c98dd0e5fa8c75e341a75f060037bd5ceae/pillow-10.4.0-cp313-cp313-manylinux_2_28_x86_64.whl", hash = "sha256:e4db64794ccdf6cb83a59d73405f63adbe2a1887012e308828596100a0b2f6cc", size = 4536767, upload-time = "2024-07-01T09:46:56.664Z" },
 630      { url = "https://files.pythonhosted.org/packages/e5/77/05fa64d1f45d12c22c314e7b97398ffb28ef2813a485465017b7978b3ce7/pillow-10.4.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:bd2880a07482090a3bcb01f4265f1936a903d70bc740bfcb1fd4e8a2ffe5cf5a", size = 4477989, upload-time = "2024-07-01T09:46:58.977Z" },
 631      { url = "https://files.pythonhosted.org/packages/12/63/b0397cfc2caae05c3fb2f4ed1b4fc4fc878f0243510a7a6034ca59726494/pillow-10.4.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:4b35b21b819ac1dbd1233317adeecd63495f6babf21b7b2512d244ff6c6ce309", size = 4610255, upload-time = "2024-07-01T09:47:01.189Z" },
 632      { url = "https://files.pythonhosted.org/packages/7b/f9/cfaa5082ca9bc4a6de66ffe1c12c2d90bf09c309a5f52b27759a596900e7/pillow-10.4.0-cp313-cp313-win32.whl", hash = "sha256:551d3fd6e9dc15e4c1eb6fc4ba2b39c0c7933fa113b220057a34f4bb3268a060", size = 2235603, upload-time = "2024-07-01T09:47:03.918Z" },
 633      { url = "https://files.pythonhosted.org/packages/01/6a/30ff0eef6e0c0e71e55ded56a38d4859bf9d3634a94a88743897b5f96936/pillow-10.4.0-cp313-cp313-win_amd64.whl", hash = "sha256:030abdbe43ee02e0de642aee345efa443740aa4d828bfe8e2eb11922ea6a21ea", size = 2554972, upload-time = "2024-07-01T09:47:06.152Z" },
 634      { url = "https://files.pythonhosted.org/packages/48/2c/2e0a52890f269435eee38b21c8218e102c621fe8d8df8b9dd06fabf879ba/pillow-10.4.0-cp313-cp313-win_arm64.whl", hash = "sha256:5b001114dd152cfd6b23befeb28d7aee43553e2402c9f159807bf55f33af8a8d", size = 2243375, upload-time = "2024-07-01T09:47:09.065Z" },
 635      { url = "https://files.pythonhosted.org/packages/56/70/f40009702a477ce87d8d9faaa4de51d6562b3445d7a314accd06e4ffb01d/pillow-10.4.0-cp38-cp38-macosx_10_10_x86_64.whl", hash = "sha256:8d4d5063501b6dd4024b8ac2f04962d661222d120381272deea52e3fc52d3736", size = 3509213, upload-time = "2024-07-01T09:47:11.662Z" },
 636      { url = "https://files.pythonhosted.org/packages/10/43/105823d233c5e5d31cea13428f4474ded9d961652307800979a59d6a4276/pillow-10.4.0-cp38-cp38-macosx_11_0_arm64.whl", hash = "sha256:7c1ee6f42250df403c5f103cbd2768a28fe1a0ea1f0f03fe151c8741e1469c8b", size = 3375883, upload-time = "2024-07-01T09:47:14.453Z" },
 637      { url = "https://files.pythonhosted.org/packages/3c/ad/7850c10bac468a20c918f6a5dbba9ecd106ea1cdc5db3c35e33a60570408/pillow-10.4.0-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b15e02e9bb4c21e39876698abf233c8c579127986f8207200bc8a8f6bb27acf2", size = 4330810, upload-time = "2024-07-01T09:47:16.695Z" },
 638      { url = "https://files.pythonhosted.org/packages/84/4c/69bbed9e436ac22f9ed193a2b64f64d68fcfbc9f4106249dc7ed4889907b/pillow-10.4.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:7a8d4bade9952ea9a77d0c3e49cbd8b2890a399422258a77f357b9cc9be8d680", size = 4444341, upload-time = "2024-07-01T09:47:19.334Z" },
 639      { url = "https://files.pythonhosted.org/packages/8f/4f/c183c63828a3f37bf09644ce94cbf72d4929b033b109160a5379c2885932/pillow-10.4.0-cp38-cp38-manylinux_2_28_aarch64.whl", hash = "sha256:43efea75eb06b95d1631cb784aa40156177bf9dd5b4b03ff38979e048258bc6b", size = 4356005, upload-time = "2024-07-01T09:47:21.805Z" },
 640      { url = "https://files.pythonhosted.org/packages/fb/ad/435fe29865f98a8fbdc64add8875a6e4f8c97749a93577a8919ec6f32c64/pillow-10.4.0-cp38-cp38-manylinux_2_28_x86_64.whl", hash = "sha256:950be4d8ba92aca4b2bb0741285a46bfae3ca699ef913ec8416c1b78eadd64cd", size = 4525201, upload-time = "2024-07-01T09:47:24.457Z" },
 641      { url = "https://files.pythonhosted.org/packages/80/74/be8bf8acdfd70e91f905a12ae13cfb2e17c0f1da745c40141e26d0971ff5/pillow-10.4.0-cp38-cp38-musllinux_1_2_aarch64.whl", hash = "sha256:d7480af14364494365e89d6fddc510a13e5a2c3584cb19ef65415ca57252fb84", size = 4460635, upload-time = "2024-07-01T09:47:26.841Z" },
 642      { url = "https://files.pythonhosted.org/packages/e4/90/763616e66dc9ad59c9b7fb58f863755e7934ef122e52349f62c7742b82d3/pillow-10.4.0-cp38-cp38-musllinux_1_2_x86_64.whl", hash = "sha256:73664fe514b34c8f02452ffb73b7a92c6774e39a647087f83d67f010eb9a0cf0", size = 4590283, upload-time = "2024-07-01T09:47:29.247Z" },
 643      { url = "https://files.pythonhosted.org/packages/69/66/03002cb5b2c27bb519cba63b9f9aa3709c6f7a5d3b285406c01f03fb77e5/pillow-10.4.0-cp38-cp38-win32.whl", hash = "sha256:e88d5e6ad0d026fba7bdab8c3f225a69f063f116462c49892b0149e21b6c0a0e", size = 2235185, upload-time = "2024-07-01T09:47:32.205Z" },
 644      { url = "https://files.pythonhosted.org/packages/f2/75/3cb820b2812405fc7feb3d0deb701ef0c3de93dc02597115e00704591bc9/pillow-10.4.0-cp38-cp38-win_amd64.whl", hash = "sha256:5161eef006d335e46895297f642341111945e2c1c899eb406882a6c61a4357ab", size = 2554594, upload-time = "2024-07-01T09:47:34.285Z" },
 645      { url = "https://files.pythonhosted.org/packages/31/85/955fa5400fa8039921f630372cfe5056eed6e1b8e0430ee4507d7de48832/pillow-10.4.0-cp39-cp39-macosx_10_10_x86_64.whl", hash = "sha256:0ae24a547e8b711ccaaf99c9ae3cd975470e1a30caa80a6aaee9a2f19c05701d", size = 3509283, upload-time = "2024-07-01T09:47:36.394Z" },
 646      { url = "https://files.pythonhosted.org/packages/23/9c/343827267eb28d41cd82b4180d33b10d868af9077abcec0af9793aa77d2d/pillow-10.4.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:298478fe4f77a4408895605f3482b6cc6222c018b2ce565c2b6b9c354ac3229b", size = 3375691, upload-time = "2024-07-01T09:47:38.853Z" },
 647      { url = "https://files.pythonhosted.org/packages/60/a3/7ebbeabcd341eab722896d1a5b59a3df98c4b4d26cf4b0385f8aa94296f7/pillow-10.4.0-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:134ace6dc392116566980ee7436477d844520a26a4b1bd4053f6f47d096997fd", size = 4328295, upload-time = "2024-07-01T09:47:41.765Z" },
 648      { url = "https://files.pythonhosted.org/packages/32/3f/c02268d0c6fb6b3958bdda673c17b315c821d97df29ae6969f20fb49388a/pillow-10.4.0-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:930044bb7679ab003b14023138b50181899da3f25de50e9dbee23b61b4de2126", size = 4440810, upload-time = "2024-07-01T09:47:44.27Z" },
 649      { url = "https://files.pythonhosted.org/packages/67/5d/1c93c8cc35f2fdd3d6cc7e4ad72d203902859a2867de6ad957d9b708eb8d/pillow-10.4.0-cp39-cp39-manylinux_2_28_aarch64.whl", hash = "sha256:c76e5786951e72ed3686e122d14c5d7012f16c8303a674d18cdcd6d89557fc5b", size = 4352283, upload-time = "2024-07-01T09:47:46.673Z" },
 650      { url = "https://files.pythonhosted.org/packages/bc/a8/8655557c9c7202b8abbd001f61ff36711cefaf750debcaa1c24d154ef602/pillow-10.4.0-cp39-cp39-manylinux_2_28_x86_64.whl", hash = "sha256:b2724fdb354a868ddf9a880cb84d102da914e99119211ef7ecbdc613b8c96b3c", size = 4521800, upload-time = "2024-07-01T09:47:48.813Z" },
 651      { url = "https://files.pythonhosted.org/packages/58/78/6f95797af64d137124f68af1bdaa13b5332da282b86031f6fa70cf368261/pillow-10.4.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:dbc6ae66518ab3c5847659e9988c3b60dc94ffb48ef9168656e0019a93dbf8a1", size = 4459177, upload-time = "2024-07-01T09:47:52.104Z" },
 652      { url = "https://files.pythonhosted.org/packages/8a/6d/2b3ce34f1c4266d79a78c9a51d1289a33c3c02833fe294ef0dcbb9cba4ed/pillow-10.4.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:06b2f7898047ae93fad74467ec3d28fe84f7831370e3c258afa533f81ef7f3df", size = 4589079, upload-time = "2024-07-01T09:47:54.999Z" },
 653      { url = "https://files.pythonhosted.org/packages/e3/e0/456258c74da1ff5bf8ef1eab06a95ca994d8b9ed44c01d45c3f8cbd1db7e/pillow-10.4.0-cp39-cp39-win32.whl", hash = "sha256:7970285ab628a3779aecc35823296a7869f889b8329c16ad5a71e4901a3dc4ef", size = 2235247, upload-time = "2024-07-01T09:47:57.666Z" },
 654      { url = "https://files.pythonhosted.org/packages/37/f8/bef952bdb32aa53741f58bf21798642209e994edc3f6598f337f23d5400a/pillow-10.4.0-cp39-cp39-win_amd64.whl", hash = "sha256:961a7293b2457b405967af9c77dcaa43cc1a8cd50d23c532e62d48ab6cdd56f5", size = 2554479, upload-time = "2024-07-01T09:47:59.881Z" },
 655      { url = "https://files.pythonhosted.org/packages/bb/8e/805201619cad6651eef5fc1fdef913804baf00053461522fabbc5588ea12/pillow-10.4.0-cp39-cp39-win_arm64.whl", hash = "sha256:32cda9e3d601a52baccb2856b8ea1fc213c90b340c542dcef77140dfa3278a9e", size = 2243226, upload-time = "2024-07-01T09:48:02.508Z" },
 656      { url = "https://files.pythonhosted.org/packages/38/30/095d4f55f3a053392f75e2eae45eba3228452783bab3d9a920b951ac495c/pillow-10.4.0-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:5b4815f2e65b30f5fbae9dfffa8636d992d49705723fe86a3661806e069352d4", size = 3493889, upload-time = "2024-07-01T09:48:04.815Z" },
 657      { url = "https://files.pythonhosted.org/packages/f3/e8/4ff79788803a5fcd5dc35efdc9386af153569853767bff74540725b45863/pillow-10.4.0-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:8f0aef4ef59694b12cadee839e2ba6afeab89c0f39a3adc02ed51d109117b8da", size = 3346160, upload-time = "2024-07-01T09:48:07.206Z" },
 658      { url = "https://files.pythonhosted.org/packages/d7/ac/4184edd511b14f760c73f5bb8a5d6fd85c591c8aff7c2229677a355c4179/pillow-10.4.0-pp310-pypy310_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9f4727572e2918acaa9077c919cbbeb73bd2b3ebcfe033b72f858fc9fbef0026", size = 3435020, upload-time = "2024-07-01T09:48:09.66Z" },
 659      { url = "https://files.pythonhosted.org/packages/da/21/1749cd09160149c0a246a81d646e05f35041619ce76f6493d6a96e8d1103/pillow-10.4.0-pp310-pypy310_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:ff25afb18123cea58a591ea0244b92eb1e61a1fd497bf6d6384f09bc3262ec3e", size = 3490539, upload-time = "2024-07-01T09:48:12.529Z" },
 660      { url = "https://files.pythonhosted.org/packages/b6/f5/f71fe1888b96083b3f6dfa0709101f61fc9e972c0c8d04e9d93ccef2a045/pillow-10.4.0-pp310-pypy310_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:dc3e2db6ba09ffd7d02ae9141cfa0ae23393ee7687248d46a7507b75d610f4f5", size = 3476125, upload-time = "2024-07-01T09:48:14.891Z" },
 661      { url = "https://files.pythonhosted.org/packages/96/b9/c0362c54290a31866c3526848583a2f45a535aa9d725fd31e25d318c805f/pillow-10.4.0-pp310-pypy310_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:02a2be69f9c9b8c1e97cf2713e789d4e398c751ecfd9967c18d0ce304efbf885", size = 3579373, upload-time = "2024-07-01T09:48:17.601Z" },
 662      { url = "https://files.pythonhosted.org/packages/52/3b/ce7a01026a7cf46e5452afa86f97a5e88ca97f562cafa76570178ab56d8d/pillow-10.4.0-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:0755ffd4a0c6f267cccbae2e9903d95477ca2f77c4fcf3a3a09570001856c8a5", size = 2554661, upload-time = "2024-07-01T09:48:20.293Z" },
 663      { url = "https://files.pythonhosted.org/packages/e1/1f/5a9fcd6ced51633c22481417e11b1b47d723f64fb536dfd67c015eb7f0ab/pillow-10.4.0-pp39-pypy39_pp73-macosx_10_15_x86_64.whl", hash = "sha256:a02364621fe369e06200d4a16558e056fe2805d3468350df3aef21e00d26214b", size = 3493850, upload-time = "2024-07-01T09:48:23.03Z" },
 664      { url = "https://files.pythonhosted.org/packages/cb/e6/3ea4755ed5320cb62aa6be2f6de47b058c6550f752dd050e86f694c59798/pillow-10.4.0-pp39-pypy39_pp73-macosx_11_0_arm64.whl", hash = "sha256:1b5dea9831a90e9d0721ec417a80d4cbd7022093ac38a568db2dd78363b00908", size = 3346118, upload-time = "2024-07-01T09:48:25.256Z" },
 665      { url = "https://files.pythonhosted.org/packages/0a/22/492f9f61e4648422b6ca39268ec8139277a5b34648d28f400faac14e0f48/pillow-10.4.0-pp39-pypy39_pp73-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:9b885f89040bb8c4a1573566bbb2f44f5c505ef6e74cec7ab9068c900047f04b", size = 3434958, upload-time = "2024-07-01T09:48:28.078Z" },
 666      { url = "https://files.pythonhosted.org/packages/f9/19/559a48ad4045704bb0547965b9a9345f5cd461347d977a56d178db28819e/pillow-10.4.0-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:87dd88ded2e6d74d31e1e0a99a726a6765cda32d00ba72dc37f0651f306daaa8", size = 3490340, upload-time = "2024-07-01T09:48:30.734Z" },
 667      { url = "https://files.pythonhosted.org/packages/d9/de/cebaca6fb79905b3a1aa0281d238769df3fb2ede34fd7c0caa286575915a/pillow-10.4.0-pp39-pypy39_pp73-manylinux_2_28_aarch64.whl", hash = "sha256:2db98790afc70118bd0255c2eeb465e9767ecf1f3c25f9a1abb8ffc8cfd1fe0a", size = 3476048, upload-time = "2024-07-01T09:48:33.292Z" },
 668      { url = "https://files.pythonhosted.org/packages/71/f0/86d5b2f04693b0116a01d75302b0a307800a90d6c351a8aa4f8ae76cd499/pillow-10.4.0-pp39-pypy39_pp73-manylinux_2_28_x86_64.whl", hash = "sha256:f7baece4ce06bade126fb84b8af1c33439a76d8a6fd818970215e0560ca28c27", size = 3579366, upload-time = "2024-07-01T09:48:36.527Z" },
 669      { url = "https://files.pythonhosted.org/packages/37/ae/2dbfc38cc4fd14aceea14bc440d5151b21f64c4c3ba3f6f4191610b7ee5d/pillow-10.4.0-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:cfdd747216947628af7b259d274771d84db2268ca062dd5faf373639d00113a3", size = 2554652, upload-time = "2024-07-01T09:48:38.789Z" },
 670  ]
 671  
 672  [[package]]
 673  name = "pillow"
 674  version = "11.3.0"
 675  source = { registry = "https://pypi.org/simple" }
 676  resolution-markers = [
 677      "python_full_version == '3.9.*'",
 678  ]
 679  sdist = { url = "https://files.pythonhosted.org/packages/f3/0d/d0d6dea55cd152ce3d6767bb38a8fc10e33796ba4ba210cbab9354b6d238/pillow-11.3.0.tar.gz", hash = "sha256:3828ee7586cd0b2091b6209e5ad53e20d0649bbe87164a459d0676e035e8f523", size = 47113069, upload-time = "2025-07-01T09:16:30.666Z" }
 680  wheels = [
 681      { url = "https://files.pythonhosted.org/packages/4c/5d/45a3553a253ac8763f3561371432a90bdbe6000fbdcf1397ffe502aa206c/pillow-11.3.0-cp310-cp310-macosx_10_10_x86_64.whl", hash = "sha256:1b9c17fd4ace828b3003dfd1e30bff24863e0eb59b535e8f80194d9cc7ecf860", size = 5316554, upload-time = "2025-07-01T09:13:39.342Z" },
 682      { url = "https://files.pythonhosted.org/packages/7c/c8/67c12ab069ef586a25a4a79ced553586748fad100c77c0ce59bb4983ac98/pillow-11.3.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:65dc69160114cdd0ca0f35cb434633c75e8e7fad4cf855177a05bf38678f73ad", size = 4686548, upload-time = "2025-07-01T09:13:41.835Z" },
 683      { url = "https://files.pythonhosted.org/packages/2f/bd/6741ebd56263390b382ae4c5de02979af7f8bd9807346d068700dd6d5cf9/pillow-11.3.0-cp310-cp310-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:7107195ddc914f656c7fc8e4a5e1c25f32e9236ea3ea860f257b0436011fddd0", size = 5859742, upload-time = "2025-07-03T13:09:47.439Z" },
 684      { url = "https://files.pythonhosted.org/packages/ca/0b/c412a9e27e1e6a829e6ab6c2dca52dd563efbedf4c9c6aa453d9a9b77359/pillow-11.3.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:cc3e831b563b3114baac7ec2ee86819eb03caa1a2cef0b481a5675b59c4fe23b", size = 7633087, upload-time = "2025-07-03T13:09:51.796Z" },
 685      { url = "https://files.pythonhosted.org/packages/59/9d/9b7076aaf30f5dd17e5e5589b2d2f5a5d7e30ff67a171eb686e4eecc2adf/pillow-11.3.0-cp310-cp310-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:f1f182ebd2303acf8c380a54f615ec883322593320a9b00438eb842c1f37ae50", size = 5963350, upload-time = "2025-07-01T09:13:43.865Z" },
 686      { url = "https://files.pythonhosted.org/packages/f0/16/1a6bf01fb622fb9cf5c91683823f073f053005c849b1f52ed613afcf8dae/pillow-11.3.0-cp310-cp310-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:4445fa62e15936a028672fd48c4c11a66d641d2c05726c7ec1f8ba6a572036ae", size = 6631840, upload-time = "2025-07-01T09:13:46.161Z" },
 687      { url = "https://files.pythonhosted.org/packages/7b/e6/6ff7077077eb47fde78739e7d570bdcd7c10495666b6afcd23ab56b19a43/pillow-11.3.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:71f511f6b3b91dd543282477be45a033e4845a40278fa8dcdbfdb07109bf18f9", size = 6074005, upload-time = "2025-07-01T09:13:47.829Z" },
 688      { url = "https://files.pythonhosted.org/packages/c3/3a/b13f36832ea6d279a697231658199e0a03cd87ef12048016bdcc84131601/pillow-11.3.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:040a5b691b0713e1f6cbe222e0f4f74cd233421e105850ae3b3c0ceda520f42e", size = 6708372, upload-time = "2025-07-01T09:13:52.145Z" },
 689      { url = "https://files.pythonhosted.org/packages/6c/e4/61b2e1a7528740efbc70b3d581f33937e38e98ef3d50b05007267a55bcb2/pillow-11.3.0-cp310-cp310-win32.whl", hash = "sha256:89bd777bc6624fe4115e9fac3352c79ed60f3bb18651420635f26e643e3dd1f6", size = 6277090, upload-time = "2025-07-01T09:13:53.915Z" },
 690      { url = "https://files.pythonhosted.org/packages/a9/d3/60c781c83a785d6afbd6a326ed4d759d141de43aa7365725cbcd65ce5e54/pillow-11.3.0-cp310-cp310-win_amd64.whl", hash = "sha256:19d2ff547c75b8e3ff46f4d9ef969a06c30ab2d4263a9e287733aa8b2429ce8f", size = 6985988, upload-time = "2025-07-01T09:13:55.699Z" },
 691      { url = "https://files.pythonhosted.org/packages/9f/28/4f4a0203165eefb3763939c6789ba31013a2e90adffb456610f30f613850/pillow-11.3.0-cp310-cp310-win_arm64.whl", hash = "sha256:819931d25e57b513242859ce1876c58c59dc31587847bf74cfe06b2e0cb22d2f", size = 2422899, upload-time = "2025-07-01T09:13:57.497Z" },
 692      { url = "https://files.pythonhosted.org/packages/db/26/77f8ed17ca4ffd60e1dcd220a6ec6d71210ba398cfa33a13a1cd614c5613/pillow-11.3.0-cp311-cp311-macosx_10_10_x86_64.whl", hash = "sha256:1cd110edf822773368b396281a2293aeb91c90a2db00d78ea43e7e861631b722", size = 5316531, upload-time = "2025-07-01T09:13:59.203Z" },
 693      { url = "https://files.pythonhosted.org/packages/cb/39/ee475903197ce709322a17a866892efb560f57900d9af2e55f86db51b0a5/pillow-11.3.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:9c412fddd1b77a75aa904615ebaa6001f169b26fd467b4be93aded278266b288", size = 4686560, upload-time = "2025-07-01T09:14:01.101Z" },
 694      { url = "https://files.pythonhosted.org/packages/d5/90/442068a160fd179938ba55ec8c97050a612426fae5ec0a764e345839f76d/pillow-11.3.0-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:7d1aa4de119a0ecac0a34a9c8bde33f34022e2e8f99104e47a3ca392fd60e37d", size = 5870978, upload-time = "2025-07-03T13:09:55.638Z" },
 695      { url = "https://files.pythonhosted.org/packages/13/92/dcdd147ab02daf405387f0218dcf792dc6dd5b14d2573d40b4caeef01059/pillow-11.3.0-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:91da1d88226663594e3f6b4b8c3c8d85bd504117d043740a8e0ec449087cc494", size = 7641168, upload-time = "2025-07-03T13:10:00.37Z" },
 696      { url = "https://files.pythonhosted.org/packages/6e/db/839d6ba7fd38b51af641aa904e2960e7a5644d60ec754c046b7d2aee00e5/pillow-11.3.0-cp311-cp311-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:643f189248837533073c405ec2f0bb250ba54598cf80e8c1e043381a60632f58", size = 5973053, upload-time = "2025-07-01T09:14:04.491Z" },
 697      { url = "https://files.pythonhosted.org/packages/f2/2f/d7675ecae6c43e9f12aa8d58b6012683b20b6edfbdac7abcb4e6af7a3784/pillow-11.3.0-cp311-cp311-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:106064daa23a745510dabce1d84f29137a37224831d88eb4ce94bb187b1d7e5f", size = 6640273, upload-time = "2025-07-01T09:14:06.235Z" },
 698      { url = "https://files.pythonhosted.org/packages/45/ad/931694675ede172e15b2ff03c8144a0ddaea1d87adb72bb07655eaffb654/pillow-11.3.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:cd8ff254faf15591e724dc7c4ddb6bf4793efcbe13802a4ae3e863cd300b493e", size = 6082043, upload-time = "2025-07-01T09:14:07.978Z" },
 699      { url = "https://files.pythonhosted.org/packages/3a/04/ba8f2b11fc80d2dd462d7abec16351b45ec99cbbaea4387648a44190351a/pillow-11.3.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:932c754c2d51ad2b2271fd01c3d121daaa35e27efae2a616f77bf164bc0b3e94", size = 6715516, upload-time = "2025-07-01T09:14:10.233Z" },
 700      { url = "https://files.pythonhosted.org/packages/48/59/8cd06d7f3944cc7d892e8533c56b0acb68399f640786313275faec1e3b6f/pillow-11.3.0-cp311-cp311-win32.whl", hash = "sha256:b4b8f3efc8d530a1544e5962bd6b403d5f7fe8b9e08227c6b255f98ad82b4ba0", size = 6274768, upload-time = "2025-07-01T09:14:11.921Z" },
 701      { url = "https://files.pythonhosted.org/packages/f1/cc/29c0f5d64ab8eae20f3232da8f8571660aa0ab4b8f1331da5c2f5f9a938e/pillow-11.3.0-cp311-cp311-win_amd64.whl", hash = "sha256:1a992e86b0dd7aeb1f053cd506508c0999d710a8f07b4c791c63843fc6a807ac", size = 6986055, upload-time = "2025-07-01T09:14:13.623Z" },
 702      { url = "https://files.pythonhosted.org/packages/c6/df/90bd886fabd544c25addd63e5ca6932c86f2b701d5da6c7839387a076b4a/pillow-11.3.0-cp311-cp311-win_arm64.whl", hash = "sha256:30807c931ff7c095620fe04448e2c2fc673fcbb1ffe2a7da3fb39613489b1ddd", size = 2423079, upload-time = "2025-07-01T09:14:15.268Z" },
 703      { url = "https://files.pythonhosted.org/packages/40/fe/1bc9b3ee13f68487a99ac9529968035cca2f0a51ec36892060edcc51d06a/pillow-11.3.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:fdae223722da47b024b867c1ea0be64e0df702c5e0a60e27daad39bf960dd1e4", size = 5278800, upload-time = "2025-07-01T09:14:17.648Z" },
 704      { url = "https://files.pythonhosted.org/packages/2c/32/7e2ac19b5713657384cec55f89065fb306b06af008cfd87e572035b27119/pillow-11.3.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:921bd305b10e82b4d1f5e802b6850677f965d8394203d182f078873851dada69", size = 4686296, upload-time = "2025-07-01T09:14:19.828Z" },
 705      { url = "https://files.pythonhosted.org/packages/8e/1e/b9e12bbe6e4c2220effebc09ea0923a07a6da1e1f1bfbc8d7d29a01ce32b/pillow-11.3.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:eb76541cba2f958032d79d143b98a3a6b3ea87f0959bbe256c0b5e416599fd5d", size = 5871726, upload-time = "2025-07-03T13:10:04.448Z" },
 706      { url = "https://files.pythonhosted.org/packages/8d/33/e9200d2bd7ba00dc3ddb78df1198a6e80d7669cce6c2bdbeb2530a74ec58/pillow-11.3.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:67172f2944ebba3d4a7b54f2e95c786a3a50c21b88456329314caaa28cda70f6", size = 7644652, upload-time = "2025-07-03T13:10:10.391Z" },
 707      { url = "https://files.pythonhosted.org/packages/41/f1/6f2427a26fc683e00d985bc391bdd76d8dd4e92fac33d841127eb8fb2313/pillow-11.3.0-cp312-cp312-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:97f07ed9f56a3b9b5f49d3661dc9607484e85c67e27f3e8be2c7d28ca032fec7", size = 5977787, upload-time = "2025-07-01T09:14:21.63Z" },
 708      { url = "https://files.pythonhosted.org/packages/e4/c9/06dd4a38974e24f932ff5f98ea3c546ce3f8c995d3f0985f8e5ba48bba19/pillow-11.3.0-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:676b2815362456b5b3216b4fd5bd89d362100dc6f4945154ff172e206a22c024", size = 6645236, upload-time = "2025-07-01T09:14:23.321Z" },
 709      { url = "https://files.pythonhosted.org/packages/40/e7/848f69fb79843b3d91241bad658e9c14f39a32f71a301bcd1d139416d1be/pillow-11.3.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:3e184b2f26ff146363dd07bde8b711833d7b0202e27d13540bfe2e35a323a809", size = 6086950, upload-time = "2025-07-01T09:14:25.237Z" },
 710      { url = "https://files.pythonhosted.org/packages/0b/1a/7cff92e695a2a29ac1958c2a0fe4c0b2393b60aac13b04a4fe2735cad52d/pillow-11.3.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:6be31e3fc9a621e071bc17bb7de63b85cbe0bfae91bb0363c893cbe67247780d", size = 6723358, upload-time = "2025-07-01T09:14:27.053Z" },
 711      { url = "https://files.pythonhosted.org/packages/26/7d/73699ad77895f69edff76b0f332acc3d497f22f5d75e5360f78cbcaff248/pillow-11.3.0-cp312-cp312-win32.whl", hash = "sha256:7b161756381f0918e05e7cb8a371fff367e807770f8fe92ecb20d905d0e1c149", size = 6275079, upload-time = "2025-07-01T09:14:30.104Z" },
 712      { url = "https://files.pythonhosted.org/packages/8c/ce/e7dfc873bdd9828f3b6e5c2bbb74e47a98ec23cc5c74fc4e54462f0d9204/pillow-11.3.0-cp312-cp312-win_amd64.whl", hash = "sha256:a6444696fce635783440b7f7a9fc24b3ad10a9ea3f0ab66c5905be1c19ccf17d", size = 6986324, upload-time = "2025-07-01T09:14:31.899Z" },
 713      { url = "https://files.pythonhosted.org/packages/16/8f/b13447d1bf0b1f7467ce7d86f6e6edf66c0ad7cf44cf5c87a37f9bed9936/pillow-11.3.0-cp312-cp312-win_arm64.whl", hash = "sha256:2aceea54f957dd4448264f9bf40875da0415c83eb85f55069d89c0ed436e3542", size = 2423067, upload-time = "2025-07-01T09:14:33.709Z" },
 714      { url = "https://files.pythonhosted.org/packages/1e/93/0952f2ed8db3a5a4c7a11f91965d6184ebc8cd7cbb7941a260d5f018cd2d/pillow-11.3.0-cp313-cp313-ios_13_0_arm64_iphoneos.whl", hash = "sha256:1c627742b539bba4309df89171356fcb3cc5a9178355b2727d1b74a6cf155fbd", size = 2128328, upload-time = "2025-07-01T09:14:35.276Z" },
 715      { url = "https://files.pythonhosted.org/packages/4b/e8/100c3d114b1a0bf4042f27e0f87d2f25e857e838034e98ca98fe7b8c0a9c/pillow-11.3.0-cp313-cp313-ios_13_0_arm64_iphonesimulator.whl", hash = "sha256:30b7c02f3899d10f13d7a48163c8969e4e653f8b43416d23d13d1bbfdc93b9f8", size = 2170652, upload-time = "2025-07-01T09:14:37.203Z" },
 716      { url = "https://files.pythonhosted.org/packages/aa/86/3f758a28a6e381758545f7cdb4942e1cb79abd271bea932998fc0db93cb6/pillow-11.3.0-cp313-cp313-ios_13_0_x86_64_iphonesimulator.whl", hash = "sha256:7859a4cc7c9295f5838015d8cc0a9c215b77e43d07a25e460f35cf516df8626f", size = 2227443, upload-time = "2025-07-01T09:14:39.344Z" },
 717      { url = "https://files.pythonhosted.org/packages/01/f4/91d5b3ffa718df2f53b0dc109877993e511f4fd055d7e9508682e8aba092/pillow-11.3.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:ec1ee50470b0d050984394423d96325b744d55c701a439d2bd66089bff963d3c", size = 5278474, upload-time = "2025-07-01T09:14:41.843Z" },
 718      { url = "https://files.pythonhosted.org/packages/f9/0e/37d7d3eca6c879fbd9dba21268427dffda1ab00d4eb05b32923d4fbe3b12/pillow-11.3.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:7db51d222548ccfd274e4572fdbf3e810a5e66b00608862f947b163e613b67dd", size = 4686038, upload-time = "2025-07-01T09:14:44.008Z" },
 719      { url = "https://files.pythonhosted.org/packages/ff/b0/3426e5c7f6565e752d81221af9d3676fdbb4f352317ceafd42899aaf5d8a/pillow-11.3.0-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:2d6fcc902a24ac74495df63faad1884282239265c6839a0a6416d33faedfae7e", size = 5864407, upload-time = "2025-07-03T13:10:15.628Z" },
 720      { url = "https://files.pythonhosted.org/packages/fc/c1/c6c423134229f2a221ee53f838d4be9d82bab86f7e2f8e75e47b6bf6cd77/pillow-11.3.0-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:f0f5d8f4a08090c6d6d578351a2b91acf519a54986c055af27e7a93feae6d3f1", size = 7639094, upload-time = "2025-07-03T13:10:21.857Z" },
 721      { url = "https://files.pythonhosted.org/packages/ba/c9/09e6746630fe6372c67c648ff9deae52a2bc20897d51fa293571977ceb5d/pillow-11.3.0-cp313-cp313-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:c37d8ba9411d6003bba9e518db0db0c58a680ab9fe5179f040b0463644bc9805", size = 5973503, upload-time = "2025-07-01T09:14:45.698Z" },
 722      { url = "https://files.pythonhosted.org/packages/d5/1c/a2a29649c0b1983d3ef57ee87a66487fdeb45132df66ab30dd37f7dbe162/pillow-11.3.0-cp313-cp313-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:13f87d581e71d9189ab21fe0efb5a23e9f28552d5be6979e84001d3b8505abe8", size = 6642574, upload-time = "2025-07-01T09:14:47.415Z" },
 723      { url = "https://files.pythonhosted.org/packages/36/de/d5cc31cc4b055b6c6fd990e3e7f0f8aaf36229a2698501bcb0cdf67c7146/pillow-11.3.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:023f6d2d11784a465f09fd09a34b150ea4672e85fb3d05931d89f373ab14abb2", size = 6084060, upload-time = "2025-07-01T09:14:49.636Z" },
 724      { url = "https://files.pythonhosted.org/packages/d5/ea/502d938cbaeec836ac28a9b730193716f0114c41325db428e6b280513f09/pillow-11.3.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:45dfc51ac5975b938e9809451c51734124e73b04d0f0ac621649821a63852e7b", size = 6721407, upload-time = "2025-07-01T09:14:51.962Z" },
 725      { url = "https://files.pythonhosted.org/packages/45/9c/9c5e2a73f125f6cbc59cc7087c8f2d649a7ae453f83bd0362ff7c9e2aee2/pillow-11.3.0-cp313-cp313-win32.whl", hash = "sha256:a4d336baed65d50d37b88ca5b60c0fa9d81e3a87d4a7930d3880d1624d5b31f3", size = 6273841, upload-time = "2025-07-01T09:14:54.142Z" },
 726      { url = "https://files.pythonhosted.org/packages/23/85/397c73524e0cd212067e0c969aa245b01d50183439550d24d9f55781b776/pillow-11.3.0-cp313-cp313-win_amd64.whl", hash = "sha256:0bce5c4fd0921f99d2e858dc4d4d64193407e1b99478bc5cacecba2311abde51", size = 6978450, upload-time = "2025-07-01T09:14:56.436Z" },
 727      { url = "https://files.pythonhosted.org/packages/17/d2/622f4547f69cd173955194b78e4d19ca4935a1b0f03a302d655c9f6aae65/pillow-11.3.0-cp313-cp313-win_arm64.whl", hash = "sha256:1904e1264881f682f02b7f8167935cce37bc97db457f8e7849dc3a6a52b99580", size = 2423055, upload-time = "2025-07-01T09:14:58.072Z" },
 728      { url = "https://files.pythonhosted.org/packages/dd/80/a8a2ac21dda2e82480852978416cfacd439a4b490a501a288ecf4fe2532d/pillow-11.3.0-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:4c834a3921375c48ee6b9624061076bc0a32a60b5532b322cc0ea64e639dd50e", size = 5281110, upload-time = "2025-07-01T09:14:59.79Z" },
 729      { url = "https://files.pythonhosted.org/packages/44/d6/b79754ca790f315918732e18f82a8146d33bcd7f4494380457ea89eb883d/pillow-11.3.0-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:5e05688ccef30ea69b9317a9ead994b93975104a677a36a8ed8106be9260aa6d", size = 4689547, upload-time = "2025-07-01T09:15:01.648Z" },
 730      { url = "https://files.pythonhosted.org/packages/49/20/716b8717d331150cb00f7fdd78169c01e8e0c219732a78b0e59b6bdb2fd6/pillow-11.3.0-cp313-cp313t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:1019b04af07fc0163e2810167918cb5add8d74674b6267616021ab558dc98ced", size = 5901554, upload-time = "2025-07-03T13:10:27.018Z" },
 731      { url = "https://files.pythonhosted.org/packages/74/cf/a9f3a2514a65bb071075063a96f0a5cf949c2f2fce683c15ccc83b1c1cab/pillow-11.3.0-cp313-cp313t-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:f944255db153ebb2b19c51fe85dd99ef0ce494123f21b9db4877ffdfc5590c7c", size = 7669132, upload-time = "2025-07-03T13:10:33.01Z" },
 732      { url = "https://files.pythonhosted.org/packages/98/3c/da78805cbdbee9cb43efe8261dd7cc0b4b93f2ac79b676c03159e9db2187/pillow-11.3.0-cp313-cp313t-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:1f85acb69adf2aaee8b7da124efebbdb959a104db34d3a2cb0f3793dbae422a8", size = 6005001, upload-time = "2025-07-01T09:15:03.365Z" },
 733      { url = "https://files.pythonhosted.org/packages/6c/fa/ce044b91faecf30e635321351bba32bab5a7e034c60187fe9698191aef4f/pillow-11.3.0-cp313-cp313t-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:05f6ecbeff5005399bb48d198f098a9b4b6bdf27b8487c7f38ca16eeb070cd59", size = 6668814, upload-time = "2025-07-01T09:15:05.655Z" },
 734      { url = "https://files.pythonhosted.org/packages/7b/51/90f9291406d09bf93686434f9183aba27b831c10c87746ff49f127ee80cb/pillow-11.3.0-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:a7bc6e6fd0395bc052f16b1a8670859964dbd7003bd0af2ff08342eb6e442cfe", size = 6113124, upload-time = "2025-07-01T09:15:07.358Z" },
 735      { url = "https://files.pythonhosted.org/packages/cd/5a/6fec59b1dfb619234f7636d4157d11fb4e196caeee220232a8d2ec48488d/pillow-11.3.0-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:83e1b0161c9d148125083a35c1c5a89db5b7054834fd4387499e06552035236c", size = 6747186, upload-time = "2025-07-01T09:15:09.317Z" },
 736      { url = "https://files.pythonhosted.org/packages/49/6b/00187a044f98255225f172de653941e61da37104a9ea60e4f6887717e2b5/pillow-11.3.0-cp313-cp313t-win32.whl", hash = "sha256:2a3117c06b8fb646639dce83694f2f9eac405472713fcb1ae887469c0d4f6788", size = 6277546, upload-time = "2025-07-01T09:15:11.311Z" },
 737      { url = "https://files.pythonhosted.org/packages/e8/5c/6caaba7e261c0d75bab23be79f1d06b5ad2a2ae49f028ccec801b0e853d6/pillow-11.3.0-cp313-cp313t-win_amd64.whl", hash = "sha256:857844335c95bea93fb39e0fa2726b4d9d758850b34075a7e3ff4f4fa3aa3b31", size = 6985102, upload-time = "2025-07-01T09:15:13.164Z" },
 738      { url = "https://files.pythonhosted.org/packages/f3/7e/b623008460c09a0cb38263c93b828c666493caee2eb34ff67f778b87e58c/pillow-11.3.0-cp313-cp313t-win_arm64.whl", hash = "sha256:8797edc41f3e8536ae4b10897ee2f637235c94f27404cac7297f7b607dd0716e", size = 2424803, upload-time = "2025-07-01T09:15:15.695Z" },
 739      { url = "https://files.pythonhosted.org/packages/73/f4/04905af42837292ed86cb1b1dabe03dce1edc008ef14c473c5c7e1443c5d/pillow-11.3.0-cp314-cp314-macosx_10_13_x86_64.whl", hash = "sha256:d9da3df5f9ea2a89b81bb6087177fb1f4d1c7146d583a3fe5c672c0d94e55e12", size = 5278520, upload-time = "2025-07-01T09:15:17.429Z" },
 740      { url = "https://files.pythonhosted.org/packages/41/b0/33d79e377a336247df6348a54e6d2a2b85d644ca202555e3faa0cf811ecc/pillow-11.3.0-cp314-cp314-macosx_11_0_arm64.whl", hash = "sha256:0b275ff9b04df7b640c59ec5a3cb113eefd3795a8df80bac69646ef699c6981a", size = 4686116, upload-time = "2025-07-01T09:15:19.423Z" },
 741      { url = "https://files.pythonhosted.org/packages/49/2d/ed8bc0ab219ae8768f529597d9509d184fe8a6c4741a6864fea334d25f3f/pillow-11.3.0-cp314-cp314-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:0743841cabd3dba6a83f38a92672cccbd69af56e3e91777b0ee7f4dba4385632", size = 5864597, upload-time = "2025-07-03T13:10:38.404Z" },
 742      { url = "https://files.pythonhosted.org/packages/b5/3d/b932bb4225c80b58dfadaca9d42d08d0b7064d2d1791b6a237f87f661834/pillow-11.3.0-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:2465a69cf967b8b49ee1b96d76718cd98c4e925414ead59fdf75cf0fd07df673", size = 7638246, upload-time = "2025-07-03T13:10:44.987Z" },
 743      { url = "https://files.pythonhosted.org/packages/09/b5/0487044b7c096f1b48f0d7ad416472c02e0e4bf6919541b111efd3cae690/pillow-11.3.0-cp314-cp314-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:41742638139424703b4d01665b807c6468e23e699e8e90cffefe291c5832b027", size = 5973336, upload-time = "2025-07-01T09:15:21.237Z" },
 744      { url = "https://files.pythonhosted.org/packages/a8/2d/524f9318f6cbfcc79fbc004801ea6b607ec3f843977652fdee4857a7568b/pillow-11.3.0-cp314-cp314-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:93efb0b4de7e340d99057415c749175e24c8864302369e05914682ba642e5d77", size = 6642699, upload-time = "2025-07-01T09:15:23.186Z" },
 745      { url = "https://files.pythonhosted.org/packages/6f/d2/a9a4f280c6aefedce1e8f615baaa5474e0701d86dd6f1dede66726462bbd/pillow-11.3.0-cp314-cp314-musllinux_1_2_aarch64.whl", hash = "sha256:7966e38dcd0fa11ca390aed7c6f20454443581d758242023cf36fcb319b1a874", size = 6083789, upload-time = "2025-07-01T09:15:25.1Z" },
 746      { url = "https://files.pythonhosted.org/packages/fe/54/86b0cd9dbb683a9d5e960b66c7379e821a19be4ac5810e2e5a715c09a0c0/pillow-11.3.0-cp314-cp314-musllinux_1_2_x86_64.whl", hash = "sha256:98a9afa7b9007c67ed84c57c9e0ad86a6000da96eaa638e4f8abe5b65ff83f0a", size = 6720386, upload-time = "2025-07-01T09:15:27.378Z" },
 747      { url = "https://files.pythonhosted.org/packages/e7/95/88efcaf384c3588e24259c4203b909cbe3e3c2d887af9e938c2022c9dd48/pillow-11.3.0-cp314-cp314-win32.whl", hash = "sha256:02a723e6bf909e7cea0dac1b0e0310be9d7650cd66222a5f1c571455c0a45214", size = 6370911, upload-time = "2025-07-01T09:15:29.294Z" },
 748      { url = "https://files.pythonhosted.org/packages/2e/cc/934e5820850ec5eb107e7b1a72dd278140731c669f396110ebc326f2a503/pillow-11.3.0-cp314-cp314-win_amd64.whl", hash = "sha256:a418486160228f64dd9e9efcd132679b7a02a5f22c982c78b6fc7dab3fefb635", size = 7117383, upload-time = "2025-07-01T09:15:31.128Z" },
 749      { url = "https://files.pythonhosted.org/packages/d6/e9/9c0a616a71da2a5d163aa37405e8aced9a906d574b4a214bede134e731bc/pillow-11.3.0-cp314-cp314-win_arm64.whl", hash = "sha256:155658efb5e044669c08896c0c44231c5e9abcaadbc5cd3648df2f7c0b96b9a6", size = 2511385, upload-time = "2025-07-01T09:15:33.328Z" },
 750      { url = "https://files.pythonhosted.org/packages/1a/33/c88376898aff369658b225262cd4f2659b13e8178e7534df9e6e1fa289f6/pillow-11.3.0-cp314-cp314t-macosx_10_13_x86_64.whl", hash = "sha256:59a03cdf019efbfeeed910bf79c7c93255c3d54bc45898ac2a4140071b02b4ae", size = 5281129, upload-time = "2025-07-01T09:15:35.194Z" },
 751      { url = "https://files.pythonhosted.org/packages/1f/70/d376247fb36f1844b42910911c83a02d5544ebd2a8bad9efcc0f707ea774/pillow-11.3.0-cp314-cp314t-macosx_11_0_arm64.whl", hash = "sha256:f8a5827f84d973d8636e9dc5764af4f0cf2318d26744b3d902931701b0d46653", size = 4689580, upload-time = "2025-07-01T09:15:37.114Z" },
 752      { url = "https://files.pythonhosted.org/packages/eb/1c/537e930496149fbac69efd2fc4329035bbe2e5475b4165439e3be9cb183b/pillow-11.3.0-cp314-cp314t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:ee92f2fd10f4adc4b43d07ec5e779932b4eb3dbfbc34790ada5a6669bc095aa6", size = 5902860, upload-time = "2025-07-03T13:10:50.248Z" },
 753      { url = "https://files.pythonhosted.org/packages/bd/57/80f53264954dcefeebcf9dae6e3eb1daea1b488f0be8b8fef12f79a3eb10/pillow-11.3.0-cp314-cp314t-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:c96d333dcf42d01f47b37e0979b6bd73ec91eae18614864622d9b87bbd5bbf36", size = 7670694, upload-time = "2025-07-03T13:10:56.432Z" },
 754      { url = "https://files.pythonhosted.org/packages/70/ff/4727d3b71a8578b4587d9c276e90efad2d6fe0335fd76742a6da08132e8c/pillow-11.3.0-cp314-cp314t-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:4c96f993ab8c98460cd0c001447bff6194403e8b1d7e149ade5f00594918128b", size = 6005888, upload-time = "2025-07-01T09:15:39.436Z" },
 755      { url = "https://files.pythonhosted.org/packages/05/ae/716592277934f85d3be51d7256f3636672d7b1abfafdc42cf3f8cbd4b4c8/pillow-11.3.0-cp314-cp314t-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:41342b64afeba938edb034d122b2dda5db2139b9a4af999729ba8818e0056477", size = 6670330, upload-time = "2025-07-01T09:15:41.269Z" },
 756      { url = "https://files.pythonhosted.org/packages/e7/bb/7fe6cddcc8827b01b1a9766f5fdeb7418680744f9082035bdbabecf1d57f/pillow-11.3.0-cp314-cp314t-musllinux_1_2_aarch64.whl", hash = "sha256:068d9c39a2d1b358eb9f245ce7ab1b5c3246c7c8c7d9ba58cfa5b43146c06e50", size = 6114089, upload-time = "2025-07-01T09:15:43.13Z" },
 757      { url = "https://files.pythonhosted.org/packages/8b/f5/06bfaa444c8e80f1a8e4bff98da9c83b37b5be3b1deaa43d27a0db37ef84/pillow-11.3.0-cp314-cp314t-musllinux_1_2_x86_64.whl", hash = "sha256:a1bc6ba083b145187f648b667e05a2534ecc4b9f2784c2cbe3089e44868f2b9b", size = 6748206, upload-time = "2025-07-01T09:15:44.937Z" },
 758      { url = "https://files.pythonhosted.org/packages/f0/77/bc6f92a3e8e6e46c0ca78abfffec0037845800ea38c73483760362804c41/pillow-11.3.0-cp314-cp314t-win32.whl", hash = "sha256:118ca10c0d60b06d006be10a501fd6bbdfef559251ed31b794668ed569c87e12", size = 6377370, upload-time = "2025-07-01T09:15:46.673Z" },
 759      { url = "https://files.pythonhosted.org/packages/4a/82/3a721f7d69dca802befb8af08b7c79ebcab461007ce1c18bd91a5d5896f9/pillow-11.3.0-cp314-cp314t-win_amd64.whl", hash = "sha256:8924748b688aa210d79883357d102cd64690e56b923a186f35a82cbc10f997db", size = 7121500, upload-time = "2025-07-01T09:15:48.512Z" },
 760      { url = "https://files.pythonhosted.org/packages/89/c7/5572fa4a3f45740eaab6ae86fcdf7195b55beac1371ac8c619d880cfe948/pillow-11.3.0-cp314-cp314t-win_arm64.whl", hash = "sha256:79ea0d14d3ebad43ec77ad5272e6ff9bba5b679ef73375ea760261207fa8e0aa", size = 2512835, upload-time = "2025-07-01T09:15:50.399Z" },
 761      { url = "https://files.pythonhosted.org/packages/9e/8e/9c089f01677d1264ab8648352dcb7773f37da6ad002542760c80107da816/pillow-11.3.0-cp39-cp39-macosx_10_10_x86_64.whl", hash = "sha256:48d254f8a4c776de343051023eb61ffe818299eeac478da55227d96e241de53f", size = 5316478, upload-time = "2025-07-01T09:15:52.209Z" },
 762      { url = "https://files.pythonhosted.org/packages/b5/a9/5749930caf674695867eb56a581e78eb5f524b7583ff10b01b6e5048acb3/pillow-11.3.0-cp39-cp39-macosx_11_0_arm64.whl", hash = "sha256:7aee118e30a4cf54fdd873bd3a29de51e29105ab11f9aad8c32123f58c8f8081", size = 4686522, upload-time = "2025-07-01T09:15:54.162Z" },
 763      { url = "https://files.pythonhosted.org/packages/43/46/0b85b763eb292b691030795f9f6bb6fcaf8948c39413c81696a01c3577f7/pillow-11.3.0-cp39-cp39-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:23cff760a9049c502721bdb743a7cb3e03365fafcdfc2ef9784610714166e5a4", size = 5853376, upload-time = "2025-07-03T13:11:01.066Z" },
 764      { url = "https://files.pythonhosted.org/packages/5e/c6/1a230ec0067243cbd60bc2dad5dc3ab46a8a41e21c15f5c9b52b26873069/pillow-11.3.0-cp39-cp39-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:6359a3bc43f57d5b375d1ad54a0074318a0844d11b76abccf478c37c986d3cfc", size = 7626020, upload-time = "2025-07-03T13:11:06.479Z" },
 765      { url = "https://files.pythonhosted.org/packages/63/dd/f296c27ffba447bfad76c6a0c44c1ea97a90cb9472b9304c94a732e8dbfb/pillow-11.3.0-cp39-cp39-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:092c80c76635f5ecb10f3f83d76716165c96f5229addbd1ec2bdbbda7d496e06", size = 5956732, upload-time = "2025-07-01T09:15:56.111Z" },
 766      { url = "https://files.pythonhosted.org/packages/a5/a0/98a3630f0b57f77bae67716562513d3032ae70414fcaf02750279c389a9e/pillow-11.3.0-cp39-cp39-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:cadc9e0ea0a2431124cde7e1697106471fc4c1da01530e679b2391c37d3fbb3a", size = 6624404, upload-time = "2025-07-01T09:15:58.245Z" },
 767      { url = "https://files.pythonhosted.org/packages/de/e6/83dfba5646a290edd9a21964da07674409e410579c341fc5b8f7abd81620/pillow-11.3.0-cp39-cp39-musllinux_1_2_aarch64.whl", hash = "sha256:6a418691000f2a418c9135a7cf0d797c1bb7d9a485e61fe8e7722845b95ef978", size = 6067760, upload-time = "2025-07-01T09:16:00.003Z" },
 768      { url = "https://files.pythonhosted.org/packages/bc/41/15ab268fe6ee9a2bc7391e2bbb20a98d3974304ab1a406a992dcb297a370/pillow-11.3.0-cp39-cp39-musllinux_1_2_x86_64.whl", hash = "sha256:97afb3a00b65cc0804d1c7abddbf090a81eaac02768af58cbdcaaa0a931e0b6d", size = 6700534, upload-time = "2025-07-01T09:16:02.29Z" },
 769      { url = "https://files.pythonhosted.org/packages/64/79/6d4f638b288300bed727ff29f2a3cb63db054b33518a95f27724915e3fbc/pillow-11.3.0-cp39-cp39-win32.whl", hash = "sha256:ea944117a7974ae78059fcc1800e5d3295172bb97035c0c1d9345fca1419da71", size = 6277091, upload-time = "2025-07-01T09:16:04.4Z" },
 770      { url = "https://files.pythonhosted.org/packages/46/05/4106422f45a05716fd34ed21763f8ec182e8ea00af6e9cb05b93a247361a/pillow-11.3.0-cp39-cp39-win_amd64.whl", hash = "sha256:e5c5858ad8ec655450a7c7df532e9842cf8df7cc349df7225c60d5d348c8aada", size = 6986091, upload-time = "2025-07-01T09:16:06.342Z" },
 771      { url = "https://files.pythonhosted.org/packages/63/c6/287fd55c2c12761d0591549d48885187579b7c257bef0c6660755b0b59ae/pillow-11.3.0-cp39-cp39-win_arm64.whl", hash = "sha256:6abdbfd3aea42be05702a8dd98832329c167ee84400a1d1f61ab11437f1717eb", size = 2422632, upload-time = "2025-07-01T09:16:08.142Z" },
 772      { url = "https://files.pythonhosted.org/packages/6f/8b/209bd6b62ce8367f47e68a218bffac88888fdf2c9fcf1ecadc6c3ec1ebc7/pillow-11.3.0-pp310-pypy310_pp73-macosx_10_15_x86_64.whl", hash = "sha256:3cee80663f29e3843b68199b9d6f4f54bd1d4a6b59bdd91bceefc51238bcb967", size = 5270556, upload-time = "2025-07-01T09:16:09.961Z" },
 773      { url = "https://files.pythonhosted.org/packages/2e/e6/231a0b76070c2cfd9e260a7a5b504fb72da0a95279410fa7afd99d9751d6/pillow-11.3.0-pp310-pypy310_pp73-macosx_11_0_arm64.whl", hash = "sha256:b5f56c3f344f2ccaf0dd875d3e180f631dc60a51b314295a3e681fe8cf851fbe", size = 4654625, upload-time = "2025-07-01T09:16:11.913Z" },
 774      { url = "https://files.pythonhosted.org/packages/13/f4/10cf94fda33cb12765f2397fc285fa6d8eb9c29de7f3185165b702fc7386/pillow-11.3.0-pp310-pypy310_pp73-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:e67d793d180c9df62f1f40aee3accca4829d3794c95098887edc18af4b8b780c", size = 4874207, upload-time = "2025-07-03T13:11:10.201Z" },
 775      { url = "https://files.pythonhosted.org/packages/72/c9/583821097dc691880c92892e8e2d41fe0a5a3d6021f4963371d2f6d57250/pillow-11.3.0-pp310-pypy310_pp73-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:d000f46e2917c705e9fb93a3606ee4a819d1e3aa7a9b442f6444f07e77cf5e25", size = 6583939, upload-time = "2025-07-03T13:11:15.68Z" },
 776      { url = "https://files.pythonhosted.org/packages/3b/8e/5c9d410f9217b12320efc7c413e72693f48468979a013ad17fd690397b9a/pillow-11.3.0-pp310-pypy310_pp73-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:527b37216b6ac3a12d7838dc3bd75208ec57c1c6d11ef01902266a5a0c14fc27", size = 4957166, upload-time = "2025-07-01T09:16:13.74Z" },
 777      { url = "https://files.pythonhosted.org/packages/62/bb/78347dbe13219991877ffb3a91bf09da8317fbfcd4b5f9140aeae020ad71/pillow-11.3.0-pp310-pypy310_pp73-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:be5463ac478b623b9dd3937afd7fb7ab3d79dd290a28e2b6df292dc75063eb8a", size = 5581482, upload-time = "2025-07-01T09:16:16.107Z" },
 778      { url = "https://files.pythonhosted.org/packages/d9/28/1000353d5e61498aaeaaf7f1e4b49ddb05f2c6575f9d4f9f914a3538b6e1/pillow-11.3.0-pp310-pypy310_pp73-win_amd64.whl", hash = "sha256:8dc70ca24c110503e16918a658b869019126ecfe03109b754c402daff12b3d9f", size = 6984596, upload-time = "2025-07-01T09:16:18.07Z" },
 779      { url = "https://files.pythonhosted.org/packages/9e/e3/6fa84033758276fb31da12e5fb66ad747ae83b93c67af17f8c6ff4cc8f34/pillow-11.3.0-pp311-pypy311_pp73-macosx_10_15_x86_64.whl", hash = "sha256:7c8ec7a017ad1bd562f93dbd8505763e688d388cde6e4a010ae1486916e713e6", size = 5270566, upload-time = "2025-07-01T09:16:19.801Z" },
 780      { url = "https://files.pythonhosted.org/packages/5b/ee/e8d2e1ab4892970b561e1ba96cbd59c0d28cf66737fc44abb2aec3795a4e/pillow-11.3.0-pp311-pypy311_pp73-macosx_11_0_arm64.whl", hash = "sha256:9ab6ae226de48019caa8074894544af5b53a117ccb9d3b3dcb2871464c829438", size = 4654618, upload-time = "2025-07-01T09:16:21.818Z" },
 781      { url = "https://files.pythonhosted.org/packages/f2/6d/17f80f4e1f0761f02160fc433abd4109fa1548dcfdca46cfdadaf9efa565/pillow-11.3.0-pp311-pypy311_pp73-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:fe27fb049cdcca11f11a7bfda64043c37b30e6b91f10cb5bab275806c32f6ab3", size = 4874248, upload-time = "2025-07-03T13:11:20.738Z" },
 782      { url = "https://files.pythonhosted.org/packages/de/5f/c22340acd61cef960130585bbe2120e2fd8434c214802f07e8c03596b17e/pillow-11.3.0-pp311-pypy311_pp73-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:465b9e8844e3c3519a983d58b80be3f668e2a7a5db97f2784e7079fbc9f9822c", size = 6583963, upload-time = "2025-07-03T13:11:26.283Z" },
 783      { url = "https://files.pythonhosted.org/packages/31/5e/03966aedfbfcbb4d5f8aa042452d3361f325b963ebbadddac05b122e47dd/pillow-11.3.0-pp311-pypy311_pp73-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:5418b53c0d59b3824d05e029669efa023bbef0f3e92e75ec8428f3799487f361", size = 4957170, upload-time = "2025-07-01T09:16:23.762Z" },
 784      { url = "https://files.pythonhosted.org/packages/cc/2d/e082982aacc927fc2cab48e1e731bdb1643a1406acace8bed0900a61464e/pillow-11.3.0-pp311-pypy311_pp73-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:504b6f59505f08ae014f724b6207ff6222662aab5cc9542577fb084ed0676ac7", size = 5581505, upload-time = "2025-07-01T09:16:25.593Z" },
 785      { url = "https://files.pythonhosted.org/packages/34/e7/ae39f538fd6844e982063c3a5e4598b8ced43b9633baa3a85ef33af8c05c/pillow-11.3.0-pp311-pypy311_pp73-win_amd64.whl", hash = "sha256:c84d689db21a1c397d001aa08241044aa2069e7587b398c8cc63020390b1c1b8", size = 6984598, upload-time = "2025-07-01T09:16:27.732Z" },
 786  ]
 787  
 788  [[package]]
 789  name = "pillow"
 790  version = "12.0.0"
 791  source = { registry = "https://pypi.org/simple" }
 792  resolution-markers = [
 793      "python_full_version >= '3.10'",
 794  ]
 795  sdist = { url = "https://files.pythonhosted.org/packages/5a/b0/cace85a1b0c9775a9f8f5d5423c8261c858760e2466c79b2dd184638b056/pillow-12.0.0.tar.gz", hash = "sha256:87d4f8125c9988bfbed67af47dd7a953e2fc7b0cc1e7800ec6d2080d490bb353", size = 47008828, upload-time = "2025-10-15T18:24:14.008Z" }
 796  wheels = [
 797      { url = "https://files.pythonhosted.org/packages/5d/08/26e68b6b5da219c2a2cb7b563af008b53bb8e6b6fcb3fa40715fcdb2523a/pillow-12.0.0-cp310-cp310-macosx_10_10_x86_64.whl", hash = "sha256:3adfb466bbc544b926d50fe8f4a4e6abd8c6bffd28a26177594e6e9b2b76572b", size = 5289809, upload-time = "2025-10-15T18:21:27.791Z" },
 798      { url = "https://files.pythonhosted.org/packages/cb/e9/4e58fb097fb74c7b4758a680aacd558810a417d1edaa7000142976ef9d2f/pillow-12.0.0-cp310-cp310-macosx_11_0_arm64.whl", hash = "sha256:1ac11e8ea4f611c3c0147424eae514028b5e9077dd99ab91e1bd7bc33ff145e1", size = 4650606, upload-time = "2025-10-15T18:21:29.823Z" },
 799      { url = "https://files.pythonhosted.org/packages/4b/e0/1fa492aa9f77b3bc6d471c468e62bfea1823056bf7e5e4f1914d7ab2565e/pillow-12.0.0-cp310-cp310-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:d49e2314c373f4c2b39446fb1a45ed333c850e09d0c59ac79b72eb3b95397363", size = 6221023, upload-time = "2025-10-15T18:21:31.415Z" },
 800      { url = "https://files.pythonhosted.org/packages/c1/09/4de7cd03e33734ccd0c876f0251401f1314e819cbfd89a0fcb6e77927cc6/pillow-12.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:c7b2a63fd6d5246349f3d3f37b14430d73ee7e8173154461785e43036ffa96ca", size = 8024937, upload-time = "2025-10-15T18:21:33.453Z" },
 801      { url = "https://files.pythonhosted.org/packages/2e/69/0688e7c1390666592876d9d474f5e135abb4acb39dcb583c4dc5490f1aff/pillow-12.0.0-cp310-cp310-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:d64317d2587c70324b79861babb9c09f71fbb780bad212018874b2c013d8600e", size = 6334139, upload-time = "2025-10-15T18:21:35.395Z" },
 802      { url = "https://files.pythonhosted.org/packages/ed/1c/880921e98f525b9b44ce747ad1ea8f73fd7e992bafe3ca5e5644bf433dea/pillow-12.0.0-cp310-cp310-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:d77153e14b709fd8b8af6f66a3afbb9ed6e9fc5ccf0b6b7e1ced7b036a228782", size = 7026074, upload-time = "2025-10-15T18:21:37.219Z" },
 803      { url = "https://files.pythonhosted.org/packages/28/03/96f718331b19b355610ef4ebdbbde3557c726513030665071fd025745671/pillow-12.0.0-cp310-cp310-musllinux_1_2_aarch64.whl", hash = "sha256:32ed80ea8a90ee3e6fa08c21e2e091bba6eda8eccc83dbc34c95169507a91f10", size = 6448852, upload-time = "2025-10-15T18:21:39.168Z" },
 804      { url = "https://files.pythonhosted.org/packages/3a/a0/6a193b3f0cc9437b122978d2c5cbce59510ccf9a5b48825096ed7472da2f/pillow-12.0.0-cp310-cp310-musllinux_1_2_x86_64.whl", hash = "sha256:c828a1ae702fc712978bda0320ba1b9893d99be0badf2647f693cc01cf0f04fa", size = 7117058, upload-time = "2025-10-15T18:21:40.997Z" },
 805      { url = "https://files.pythonhosted.org/packages/a7/c4/043192375eaa4463254e8e61f0e2ec9a846b983929a8d0a7122e0a6d6fff/pillow-12.0.0-cp310-cp310-win32.whl", hash = "sha256:bd87e140e45399c818fac4247880b9ce719e4783d767e030a883a970be632275", size = 6295431, upload-time = "2025-10-15T18:21:42.518Z" },
 806      { url = "https://files.pythonhosted.org/packages/92/c6/c2f2fc7e56301c21827e689bb8b0b465f1b52878b57471a070678c0c33cd/pillow-12.0.0-cp310-cp310-win_amd64.whl", hash = "sha256:455247ac8a4cfb7b9bc45b7e432d10421aea9fc2e74d285ba4072688a74c2e9d", size = 7000412, upload-time = "2025-10-15T18:21:44.404Z" },
 807      { url = "https://files.pythonhosted.org/packages/b2/d2/5f675067ba82da7a1c238a73b32e3fd78d67f9d9f80fbadd33a40b9c0481/pillow-12.0.0-cp310-cp310-win_arm64.whl", hash = "sha256:6ace95230bfb7cd79ef66caa064bbe2f2a1e63d93471c3a2e1f1348d9f22d6b7", size = 2435903, upload-time = "2025-10-15T18:21:46.29Z" },
 808      { url = "https://files.pythonhosted.org/packages/0e/5a/a2f6773b64edb921a756eb0729068acad9fc5208a53f4a349396e9436721/pillow-12.0.0-cp311-cp311-macosx_10_10_x86_64.whl", hash = "sha256:0fd00cac9c03256c8b2ff58f162ebcd2587ad3e1f2e397eab718c47e24d231cc", size = 5289798, upload-time = "2025-10-15T18:21:47.763Z" },
 809      { url = "https://files.pythonhosted.org/packages/2e/05/069b1f8a2e4b5a37493da6c5868531c3f77b85e716ad7a590ef87d58730d/pillow-12.0.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:a3475b96f5908b3b16c47533daaa87380c491357d197564e0ba34ae75c0f3257", size = 4650589, upload-time = "2025-10-15T18:21:49.515Z" },
 810      { url = "https://files.pythonhosted.org/packages/61/e3/2c820d6e9a36432503ead175ae294f96861b07600a7156154a086ba7111a/pillow-12.0.0-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:110486b79f2d112cf6add83b28b627e369219388f64ef2f960fef9ebaf54c642", size = 6230472, upload-time = "2025-10-15T18:21:51.052Z" },
 811      { url = "https://files.pythonhosted.org/packages/4f/89/63427f51c64209c5e23d4d52071c8d0f21024d3a8a487737caaf614a5795/pillow-12.0.0-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:5269cc1caeedb67e6f7269a42014f381f45e2e7cd42d834ede3c703a1d915fe3", size = 8033887, upload-time = "2025-10-15T18:21:52.604Z" },
 812      { url = "https://files.pythonhosted.org/packages/f6/1b/c9711318d4901093c15840f268ad649459cd81984c9ec9887756cca049a5/pillow-12.0.0-cp311-cp311-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:aa5129de4e174daccbc59d0a3b6d20eaf24417d59851c07ebb37aeb02947987c", size = 6343964, upload-time = "2025-10-15T18:21:54.619Z" },
 813      { url = "https://files.pythonhosted.org/packages/41/1e/db9470f2d030b4995083044cd8738cdd1bf773106819f6d8ba12597d5352/pillow-12.0.0-cp311-cp311-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:bee2a6db3a7242ea309aa7ee8e2780726fed67ff4e5b40169f2c940e7eb09227", size = 7034756, upload-time = "2025-10-15T18:21:56.151Z" },
 814      { url = "https://files.pythonhosted.org/packages/cc/b0/6177a8bdd5ee4ed87cba2de5a3cc1db55ffbbec6176784ce5bb75aa96798/pillow-12.0.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:90387104ee8400a7b4598253b4c406f8958f59fcf983a6cea2b50d59f7d63d0b", size = 6458075, upload-time = "2025-10-15T18:21:57.759Z" },
 815      { url = "https://files.pythonhosted.org/packages/bc/5e/61537aa6fa977922c6a03253a0e727e6e4a72381a80d63ad8eec350684f2/pillow-12.0.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:bc91a56697869546d1b8f0a3ff35224557ae7f881050e99f615e0119bf934b4e", size = 7125955, upload-time = "2025-10-15T18:21:59.372Z" },
 816      { url = "https://files.pythonhosted.org/packages/1f/3d/d5033539344ee3cbd9a4d69e12e63ca3a44a739eb2d4c8da350a3d38edd7/pillow-12.0.0-cp311-cp311-win32.whl", hash = "sha256:27f95b12453d165099c84f8a8bfdfd46b9e4bda9e0e4b65f0635430027f55739", size = 6298440, upload-time = "2025-10-15T18:22:00.982Z" },
 817      { url = "https://files.pythonhosted.org/packages/4d/42/aaca386de5cc8bd8a0254516957c1f265e3521c91515b16e286c662854c4/pillow-12.0.0-cp311-cp311-win_amd64.whl", hash = "sha256:b583dc9070312190192631373c6c8ed277254aa6e6084b74bdd0a6d3b221608e", size = 6999256, upload-time = "2025-10-15T18:22:02.617Z" },
 818      { url = "https://files.pythonhosted.org/packages/ba/f1/9197c9c2d5708b785f631a6dfbfa8eb3fb9672837cb92ae9af812c13b4ed/pillow-12.0.0-cp311-cp311-win_arm64.whl", hash = "sha256:759de84a33be3b178a64c8ba28ad5c135900359e85fb662bc6e403ad4407791d", size = 2436025, upload-time = "2025-10-15T18:22:04.598Z" },
 819      { url = "https://files.pythonhosted.org/packages/2c/90/4fcce2c22caf044e660a198d740e7fbc14395619e3cb1abad12192c0826c/pillow-12.0.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:53561a4ddc36facb432fae7a9d8afbfaf94795414f5cdc5fc52f28c1dca90371", size = 5249377, upload-time = "2025-10-15T18:22:05.993Z" },
 820      { url = "https://files.pythonhosted.org/packages/fd/e0/ed960067543d080691d47d6938ebccbf3976a931c9567ab2fbfab983a5dd/pillow-12.0.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:71db6b4c1653045dacc1585c1b0d184004f0d7e694c7b34ac165ca70c0838082", size = 4650343, upload-time = "2025-10-15T18:22:07.718Z" },
 821      { url = "https://files.pythonhosted.org/packages/e7/a1/f81fdeddcb99c044bf7d6faa47e12850f13cee0849537a7d27eeab5534d4/pillow-12.0.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:2fa5f0b6716fc88f11380b88b31fe591a06c6315e955c096c35715788b339e3f", size = 6232981, upload-time = "2025-10-15T18:22:09.287Z" },
 822      { url = "https://files.pythonhosted.org/packages/88/e1/9098d3ce341a8750b55b0e00c03f1630d6178f38ac191c81c97a3b047b44/pillow-12.0.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:82240051c6ca513c616f7f9da06e871f61bfd7805f566275841af15015b8f98d", size = 8041399, upload-time = "2025-10-15T18:22:10.872Z" },
 823      { url = "https://files.pythonhosted.org/packages/a7/62/a22e8d3b602ae8cc01446d0c57a54e982737f44b6f2e1e019a925143771d/pillow-12.0.0-cp312-cp312-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:55f818bd74fe2f11d4d7cbc65880a843c4075e0ac7226bc1a23261dbea531953", size = 6347740, upload-time = "2025-10-15T18:22:12.769Z" },
 824      { url = "https://files.pythonhosted.org/packages/4f/87/424511bdcd02c8d7acf9f65caa09f291a519b16bd83c3fb3374b3d4ae951/pillow-12.0.0-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:b87843e225e74576437fd5b6a4c2205d422754f84a06942cfaf1dc32243e45a8", size = 7040201, upload-time = "2025-10-15T18:22:14.813Z" },
 825      { url = "https://files.pythonhosted.org/packages/dc/4d/435c8ac688c54d11755aedfdd9f29c9eeddf68d150fe42d1d3dbd2365149/pillow-12.0.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:c607c90ba67533e1b2355b821fef6764d1dd2cbe26b8c1005ae84f7aea25ff79", size = 6462334, upload-time = "2025-10-15T18:22:16.375Z" },
 826      { url = "https://files.pythonhosted.org/packages/2b/f2/ad34167a8059a59b8ad10bc5c72d4d9b35acc6b7c0877af8ac885b5f2044/pillow-12.0.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:21f241bdd5080a15bc86d3466a9f6074a9c2c2b314100dd896ac81ee6db2f1ba", size = 7134162, upload-time = "2025-10-15T18:22:17.996Z" },
 827      { url = "https://files.pythonhosted.org/packages/0c/b1/a7391df6adacf0a5c2cf6ac1cf1fcc1369e7d439d28f637a847f8803beb3/pillow-12.0.0-cp312-cp312-win32.whl", hash = "sha256:dd333073e0cacdc3089525c7df7d39b211bcdf31fc2824e49d01c6b6187b07d0", size = 6298769, upload-time = "2025-10-15T18:22:19.923Z" },
 828      { url = "https://files.pythonhosted.org/packages/a2/0b/d87733741526541c909bbf159e338dcace4f982daac6e5a8d6be225ca32d/pillow-12.0.0-cp312-cp312-win_amd64.whl", hash = "sha256:9fe611163f6303d1619bbcb653540a4d60f9e55e622d60a3108be0d5b441017a", size = 7001107, upload-time = "2025-10-15T18:22:21.644Z" },
 829      { url = "https://files.pythonhosted.org/packages/bc/96/aaa61ce33cc98421fb6088af2a03be4157b1e7e0e87087c888e2370a7f45/pillow-12.0.0-cp312-cp312-win_arm64.whl", hash = "sha256:7dfb439562f234f7d57b1ac6bc8fe7f838a4bd49c79230e0f6a1da93e82f1fad", size = 2436012, upload-time = "2025-10-15T18:22:23.621Z" },
 830      { url = "https://files.pythonhosted.org/packages/62/f2/de993bb2d21b33a98d031ecf6a978e4b61da207bef02f7b43093774c480d/pillow-12.0.0-cp313-cp313-ios_13_0_arm64_iphoneos.whl", hash = "sha256:0869154a2d0546545cde61d1789a6524319fc1897d9ee31218eae7a60ccc5643", size = 4045493, upload-time = "2025-10-15T18:22:25.758Z" },
 831      { url = "https://files.pythonhosted.org/packages/0e/b6/bc8d0c4c9f6f111a783d045310945deb769b806d7574764234ffd50bc5ea/pillow-12.0.0-cp313-cp313-ios_13_0_arm64_iphonesimulator.whl", hash = "sha256:a7921c5a6d31b3d756ec980f2f47c0cfdbce0fc48c22a39347a895f41f4a6ea4", size = 4120461, upload-time = "2025-10-15T18:22:27.286Z" },
 832      { url = "https://files.pythonhosted.org/packages/5d/57/d60d343709366a353dc56adb4ee1e7d8a2cc34e3fbc22905f4167cfec119/pillow-12.0.0-cp313-cp313-ios_13_0_x86_64_iphonesimulator.whl", hash = "sha256:1ee80a59f6ce048ae13cda1abf7fbd2a34ab9ee7d401c46be3ca685d1999a399", size = 3576912, upload-time = "2025-10-15T18:22:28.751Z" },
 833      { url = "https://files.pythonhosted.org/packages/a4/a4/a0a31467e3f83b94d37568294b01d22b43ae3c5d85f2811769b9c66389dd/pillow-12.0.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:c50f36a62a22d350c96e49ad02d0da41dbd17ddc2e29750dbdba4323f85eb4a5", size = 5249132, upload-time = "2025-10-15T18:22:30.641Z" },
 834      { url = "https://files.pythonhosted.org/packages/83/06/48eab21dd561de2914242711434c0c0eb992ed08ff3f6107a5f44527f5e9/pillow-12.0.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:5193fde9a5f23c331ea26d0cf171fbf67e3f247585f50c08b3e205c7aeb4589b", size = 4650099, upload-time = "2025-10-15T18:22:32.73Z" },
 835      { url = "https://files.pythonhosted.org/packages/fc/bd/69ed99fd46a8dba7c1887156d3572fe4484e3f031405fcc5a92e31c04035/pillow-12.0.0-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:bde737cff1a975b70652b62d626f7785e0480918dece11e8fef3c0cf057351c3", size = 6230808, upload-time = "2025-10-15T18:22:34.337Z" },
 836      { url = "https://files.pythonhosted.org/packages/ea/94/8fad659bcdbf86ed70099cb60ae40be6acca434bbc8c4c0d4ef356d7e0de/pillow-12.0.0-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:a6597ff2b61d121172f5844b53f21467f7082f5fb385a9a29c01414463f93b07", size = 8037804, upload-time = "2025-10-15T18:22:36.402Z" },
 837      { url = "https://files.pythonhosted.org/packages/20/39/c685d05c06deecfd4e2d1950e9a908aa2ca8bc4e6c3b12d93b9cafbd7837/pillow-12.0.0-cp313-cp313-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:0b817e7035ea7f6b942c13aa03bb554fc44fea70838ea21f8eb31c638326584e", size = 6345553, upload-time = "2025-10-15T18:22:38.066Z" },
 838      { url = "https://files.pythonhosted.org/packages/38/57/755dbd06530a27a5ed74f8cb0a7a44a21722ebf318edbe67ddbd7fb28f88/pillow-12.0.0-cp313-cp313-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:f4f1231b7dec408e8670264ce63e9c71409d9583dd21d32c163e25213ee2a344", size = 7037729, upload-time = "2025-10-15T18:22:39.769Z" },
 839      { url = "https://files.pythonhosted.org/packages/ca/b6/7e94f4c41d238615674d06ed677c14883103dce1c52e4af16f000338cfd7/pillow-12.0.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:6e51b71417049ad6ab14c49608b4a24d8fb3fe605e5dfabfe523b58064dc3d27", size = 6459789, upload-time = "2025-10-15T18:22:41.437Z" },
 840      { url = "https://files.pythonhosted.org/packages/9c/14/4448bb0b5e0f22dd865290536d20ec8a23b64e2d04280b89139f09a36bb6/pillow-12.0.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:d120c38a42c234dc9a8c5de7ceaaf899cf33561956acb4941653f8bdc657aa79", size = 7130917, upload-time = "2025-10-15T18:22:43.152Z" },
 841      { url = "https://files.pythonhosted.org/packages/dd/ca/16c6926cc1c015845745d5c16c9358e24282f1e588237a4c36d2b30f182f/pillow-12.0.0-cp313-cp313-win32.whl", hash = "sha256:4cc6b3b2efff105c6a1656cfe59da4fdde2cda9af1c5e0b58529b24525d0a098", size = 6302391, upload-time = "2025-10-15T18:22:44.753Z" },
 842      { url = "https://files.pythonhosted.org/packages/6d/2a/dd43dcfd6dae9b6a49ee28a8eedb98c7d5ff2de94a5d834565164667b97b/pillow-12.0.0-cp313-cp313-win_amd64.whl", hash = "sha256:4cf7fed4b4580601c4345ceb5d4cbf5a980d030fd5ad07c4d2ec589f95f09905", size = 7007477, upload-time = "2025-10-15T18:22:46.838Z" },
 843      { url = "https://files.pythonhosted.org/packages/77/f0/72ea067f4b5ae5ead653053212af05ce3705807906ba3f3e8f58ddf617e6/pillow-12.0.0-cp313-cp313-win_arm64.whl", hash = "sha256:9f0b04c6b8584c2c193babcccc908b38ed29524b29dd464bc8801bf10d746a3a", size = 2435918, upload-time = "2025-10-15T18:22:48.399Z" },
 844      { url = "https://files.pythonhosted.org/packages/f5/5e/9046b423735c21f0487ea6cb5b10f89ea8f8dfbe32576fe052b5ba9d4e5b/pillow-12.0.0-cp313-cp313t-macosx_10_13_x86_64.whl", hash = "sha256:7fa22993bac7b77b78cae22bad1e2a987ddf0d9015c63358032f84a53f23cdc3", size = 5251406, upload-time = "2025-10-15T18:22:49.905Z" },
 845      { url = "https://files.pythonhosted.org/packages/12/66/982ceebcdb13c97270ef7a56c3969635b4ee7cd45227fa707c94719229c5/pillow-12.0.0-cp313-cp313t-macosx_11_0_arm64.whl", hash = "sha256:f135c702ac42262573fe9714dfe99c944b4ba307af5eb507abef1667e2cbbced", size = 4653218, upload-time = "2025-10-15T18:22:51.587Z" },
 846      { url = "https://files.pythonhosted.org/packages/16/b3/81e625524688c31859450119bf12674619429cab3119eec0e30a7a1029cb/pillow-12.0.0-cp313-cp313t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:c85de1136429c524e55cfa4e033b4a7940ac5c8ee4d9401cc2d1bf48154bbc7b", size = 6266564, upload-time = "2025-10-15T18:22:53.215Z" },
 847      { url = "https://files.pythonhosted.org/packages/98/59/dfb38f2a41240d2408096e1a76c671d0a105a4a8471b1871c6902719450c/pillow-12.0.0-cp313-cp313t-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:38df9b4bfd3db902c9c2bd369bcacaf9d935b2fff73709429d95cc41554f7b3d", size = 8069260, upload-time = "2025-10-15T18:22:54.933Z" },
 848      { url = "https://files.pythonhosted.org/packages/dc/3d/378dbea5cd1874b94c312425ca77b0f47776c78e0df2df751b820c8c1d6c/pillow-12.0.0-cp313-cp313t-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:7d87ef5795da03d742bf49439f9ca4d027cde49c82c5371ba52464aee266699a", size = 6379248, upload-time = "2025-10-15T18:22:56.605Z" },
 849      { url = "https://files.pythonhosted.org/packages/84/b0/d525ef47d71590f1621510327acec75ae58c721dc071b17d8d652ca494d8/pillow-12.0.0-cp313-cp313t-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:aff9e4d82d082ff9513bdd6acd4f5bd359f5b2c870907d2b0a9c5e10d40c88fe", size = 7066043, upload-time = "2025-10-15T18:22:58.53Z" },
 850      { url = "https://files.pythonhosted.org/packages/61/2c/aced60e9cf9d0cde341d54bf7932c9ffc33ddb4a1595798b3a5150c7ec4e/pillow-12.0.0-cp313-cp313t-musllinux_1_2_aarch64.whl", hash = "sha256:8d8ca2b210ada074d57fcee40c30446c9562e542fc46aedc19baf758a93532ee", size = 6490915, upload-time = "2025-10-15T18:23:00.582Z" },
 851      { url = "https://files.pythonhosted.org/packages/ef/26/69dcb9b91f4e59f8f34b2332a4a0a951b44f547c4ed39d3e4dcfcff48f89/pillow-12.0.0-cp313-cp313t-musllinux_1_2_x86_64.whl", hash = "sha256:99a7f72fb6249302aa62245680754862a44179b545ded638cf1fef59befb57ef", size = 7157998, upload-time = "2025-10-15T18:23:02.627Z" },
 852      { url = "https://files.pythonhosted.org/packages/61/2b/726235842220ca95fa441ddf55dd2382b52ab5b8d9c0596fe6b3f23dafe8/pillow-12.0.0-cp313-cp313t-win32.whl", hash = "sha256:4078242472387600b2ce8d93ade8899c12bf33fa89e55ec89fe126e9d6d5d9e9", size = 6306201, upload-time = "2025-10-15T18:23:04.709Z" },
 853      { url = "https://files.pythonhosted.org/packages/c0/3d/2afaf4e840b2df71344ababf2f8edd75a705ce500e5dc1e7227808312ae1/pillow-12.0.0-cp313-cp313t-win_amd64.whl", hash = "sha256:2c54c1a783d6d60595d3514f0efe9b37c8808746a66920315bfd34a938d7994b", size = 7013165, upload-time = "2025-10-15T18:23:06.46Z" },
 854      { url = "https://files.pythonhosted.org/packages/6f/75/3fa09aa5cf6ed04bee3fa575798ddf1ce0bace8edb47249c798077a81f7f/pillow-12.0.0-cp313-cp313t-win_arm64.whl", hash = "sha256:26d9f7d2b604cd23aba3e9faf795787456ac25634d82cd060556998e39c6fa47", size = 2437834, upload-time = "2025-10-15T18:23:08.194Z" },
 855      { url = "https://files.pythonhosted.org/packages/54/2a/9a8c6ba2c2c07b71bec92cf63e03370ca5e5f5c5b119b742bcc0cde3f9c5/pillow-12.0.0-cp314-cp314-ios_13_0_arm64_iphoneos.whl", hash = "sha256:beeae3f27f62308f1ddbcfb0690bf44b10732f2ef43758f169d5e9303165d3f9", size = 4045531, upload-time = "2025-10-15T18:23:10.121Z" },
 856      { url = "https://files.pythonhosted.org/packages/84/54/836fdbf1bfb3d66a59f0189ff0b9f5f666cee09c6188309300df04ad71fa/pillow-12.0.0-cp314-cp314-ios_13_0_arm64_iphonesimulator.whl", hash = "sha256:d4827615da15cd59784ce39d3388275ec093ae3ee8d7f0c089b76fa87af756c2", size = 4120554, upload-time = "2025-10-15T18:23:12.14Z" },
 857      { url = "https://files.pythonhosted.org/packages/0d/cd/16aec9f0da4793e98e6b54778a5fbce4f375c6646fe662e80600b8797379/pillow-12.0.0-cp314-cp314-ios_13_0_x86_64_iphonesimulator.whl", hash = "sha256:3e42edad50b6909089750e65c91aa09aaf1e0a71310d383f11321b27c224ed8a", size = 3576812, upload-time = "2025-10-15T18:23:13.962Z" },
 858      { url = "https://files.pythonhosted.org/packages/f6/b7/13957fda356dc46339298b351cae0d327704986337c3c69bb54628c88155/pillow-12.0.0-cp314-cp314-macosx_10_15_x86_64.whl", hash = "sha256:e5d8efac84c9afcb40914ab49ba063d94f5dbdf5066db4482c66a992f47a3a3b", size = 5252689, upload-time = "2025-10-15T18:23:15.562Z" },
 859      { url = "https://files.pythonhosted.org/packages/fc/f5/eae31a306341d8f331f43edb2e9122c7661b975433de5e447939ae61c5da/pillow-12.0.0-cp314-cp314-macosx_11_0_arm64.whl", hash = "sha256:266cd5f2b63ff316d5a1bba46268e603c9caf5606d44f38c2873c380950576ad", size = 4650186, upload-time = "2025-10-15T18:23:17.379Z" },
 860      { url = "https://files.pythonhosted.org/packages/86/62/2a88339aa40c4c77e79108facbd307d6091e2c0eb5b8d3cf4977cfca2fe6/pillow-12.0.0-cp314-cp314-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:58eea5ebe51504057dd95c5b77d21700b77615ab0243d8152793dc00eb4faf01", size = 6230308, upload-time = "2025-10-15T18:23:18.971Z" },
 861      { url = "https://files.pythonhosted.org/packages/c7/33/5425a8992bcb32d1cb9fa3dd39a89e613d09a22f2c8083b7bf43c455f760/pillow-12.0.0-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:f13711b1a5ba512d647a0e4ba79280d3a9a045aaf7e0cc6fbe96b91d4cdf6b0c", size = 8039222, upload-time = "2025-10-15T18:23:20.909Z" },
 862      { url = "https://files.pythonhosted.org/packages/d8/61/3f5d3b35c5728f37953d3eec5b5f3e77111949523bd2dd7f31a851e50690/pillow-12.0.0-cp314-cp314-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:6846bd2d116ff42cba6b646edf5bf61d37e5cbd256425fa089fee4ff5c07a99e", size = 6346657, upload-time = "2025-10-15T18:23:23.077Z" },
 863      { url = "https://files.pythonhosted.org/packages/3a/be/ee90a3d79271227e0f0a33c453531efd6ed14b2e708596ba5dd9be948da3/pillow-12.0.0-cp314-cp314-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:c98fa880d695de164b4135a52fd2e9cd7b7c90a9d8ac5e9e443a24a95ef9248e", size = 7038482, upload-time = "2025-10-15T18:23:25.005Z" },
 864      { url = "https://files.pythonhosted.org/packages/44/34/a16b6a4d1ad727de390e9bd9f19f5f669e079e5826ec0f329010ddea492f/pillow-12.0.0-cp314-cp314-musllinux_1_2_aarch64.whl", hash = "sha256:fa3ed2a29a9e9d2d488b4da81dcb54720ac3104a20bf0bd273f1e4648aff5af9", size = 6461416, upload-time = "2025-10-15T18:23:27.009Z" },
 865      { url = "https://files.pythonhosted.org/packages/b6/39/1aa5850d2ade7d7ba9f54e4e4c17077244ff7a2d9e25998c38a29749eb3f/pillow-12.0.0-cp314-cp314-musllinux_1_2_x86_64.whl", hash = "sha256:d034140032870024e6b9892c692fe2968493790dd57208b2c37e3fb35f6df3ab", size = 7131584, upload-time = "2025-10-15T18:23:29.752Z" },
 866      { url = "https://files.pythonhosted.org/packages/bf/db/4fae862f8fad0167073a7733973bfa955f47e2cac3dc3e3e6257d10fab4a/pillow-12.0.0-cp314-cp314-win32.whl", hash = "sha256:1b1b133e6e16105f524a8dec491e0586d072948ce15c9b914e41cdadd209052b", size = 6400621, upload-time = "2025-10-15T18:23:32.06Z" },
 867      { url = "https://files.pythonhosted.org/packages/2b/24/b350c31543fb0107ab2599464d7e28e6f856027aadda995022e695313d94/pillow-12.0.0-cp314-cp314-win_amd64.whl", hash = "sha256:8dc232e39d409036af549c86f24aed8273a40ffa459981146829a324e0848b4b", size = 7142916, upload-time = "2025-10-15T18:23:34.71Z" },
 868      { url = "https://files.pythonhosted.org/packages/0f/9b/0ba5a6fd9351793996ef7487c4fdbde8d3f5f75dbedc093bb598648fddf0/pillow-12.0.0-cp314-cp314-win_arm64.whl", hash = "sha256:d52610d51e265a51518692045e372a4c363056130d922a7351429ac9f27e70b0", size = 2523836, upload-time = "2025-10-15T18:23:36.967Z" },
 869      { url = "https://files.pythonhosted.org/packages/f5/7a/ceee0840aebc579af529b523d530840338ecf63992395842e54edc805987/pillow-12.0.0-cp314-cp314t-macosx_10_15_x86_64.whl", hash = "sha256:1979f4566bb96c1e50a62d9831e2ea2d1211761e5662afc545fa766f996632f6", size = 5255092, upload-time = "2025-10-15T18:23:38.573Z" },
 870      { url = "https://files.pythonhosted.org/packages/44/76/20776057b4bfd1aef4eeca992ebde0f53a4dce874f3ae693d0ec90a4f79b/pillow-12.0.0-cp314-cp314t-macosx_11_0_arm64.whl", hash = "sha256:b2e4b27a6e15b04832fe9bf292b94b5ca156016bbc1ea9c2c20098a0320d6cf6", size = 4653158, upload-time = "2025-10-15T18:23:40.238Z" },
 871      { url = "https://files.pythonhosted.org/packages/82/3f/d9ff92ace07be8836b4e7e87e6a4c7a8318d47c2f1463ffcf121fc57d9cb/pillow-12.0.0-cp314-cp314t-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:fb3096c30df99fd01c7bf8e544f392103d0795b9f98ba71a8054bcbf56b255f1", size = 6267882, upload-time = "2025-10-15T18:23:42.434Z" },
 872      { url = "https://files.pythonhosted.org/packages/9f/7a/4f7ff87f00d3ad33ba21af78bfcd2f032107710baf8280e3722ceec28cda/pillow-12.0.0-cp314-cp314t-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:7438839e9e053ef79f7112c881cef684013855016f928b168b81ed5835f3e75e", size = 8071001, upload-time = "2025-10-15T18:23:44.29Z" },
 873      { url = "https://files.pythonhosted.org/packages/75/87/fcea108944a52dad8cca0715ae6247e271eb80459364a98518f1e4f480c1/pillow-12.0.0-cp314-cp314t-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:5d5c411a8eaa2299322b647cd932586b1427367fd3184ffbb8f7a219ea2041ca", size = 6380146, upload-time = "2025-10-15T18:23:46.065Z" },
 874      { url = "https://files.pythonhosted.org/packages/91/52/0d31b5e571ef5fd111d2978b84603fce26aba1b6092f28e941cb46570745/pillow-12.0.0-cp314-cp314t-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:d7e091d464ac59d2c7ad8e7e08105eaf9dafbc3883fd7265ffccc2baad6ac925", size = 7067344, upload-time = "2025-10-15T18:23:47.898Z" },
 875      { url = "https://files.pythonhosted.org/packages/7b/f4/2dd3d721f875f928d48e83bb30a434dee75a2531bca839bb996bb0aa5a91/pillow-12.0.0-cp314-cp314t-musllinux_1_2_aarch64.whl", hash = "sha256:792a2c0be4dcc18af9d4a2dfd8a11a17d5e25274a1062b0ec1c2d79c76f3e7f8", size = 6491864, upload-time = "2025-10-15T18:23:49.607Z" },
 876      { url = "https://files.pythonhosted.org/packages/30/4b/667dfcf3d61fc309ba5a15b141845cece5915e39b99c1ceab0f34bf1d124/pillow-12.0.0-cp314-cp314t-musllinux_1_2_x86_64.whl", hash = "sha256:afbefa430092f71a9593a99ab6a4e7538bc9eabbf7bf94f91510d3503943edc4", size = 7158911, upload-time = "2025-10-15T18:23:51.351Z" },
 877      { url = "https://files.pythonhosted.org/packages/a2/2f/16cabcc6426c32218ace36bf0d55955e813f2958afddbf1d391849fee9d1/pillow-12.0.0-cp314-cp314t-win32.whl", hash = "sha256:3830c769decf88f1289680a59d4f4c46c72573446352e2befec9a8512104fa52", size = 6408045, upload-time = "2025-10-15T18:23:53.177Z" },
 878      { url = "https://files.pythonhosted.org/packages/35/73/e29aa0c9c666cf787628d3f0dcf379f4791fba79f4936d02f8b37165bdf8/pillow-12.0.0-cp314-cp314t-win_amd64.whl", hash = "sha256:905b0365b210c73afb0ebe9101a32572152dfd1c144c7e28968a331b9217b94a", size = 7148282, upload-time = "2025-10-15T18:23:55.316Z" },
 879      { url = "https://files.pythonhosted.org/packages/c1/70/6b41bdcddf541b437bbb9f47f94d2db5d9ddef6c37ccab8c9107743748a4/pillow-12.0.0-cp314-cp314t-win_arm64.whl", hash = "sha256:99353a06902c2e43b43e8ff74ee65a7d90307d82370604746738a1e0661ccca7", size = 2525630, upload-time = "2025-10-15T18:23:57.149Z" },
 880      { url = "https://files.pythonhosted.org/packages/1d/b3/582327e6c9f86d037b63beebe981425d6811104cb443e8193824ef1a2f27/pillow-12.0.0-pp311-pypy311_pp73-macosx_10_15_x86_64.whl", hash = "sha256:b22bd8c974942477156be55a768f7aa37c46904c175be4e158b6a86e3a6b7ca8", size = 5215068, upload-time = "2025-10-15T18:23:59.594Z" },
 881      { url = "https://files.pythonhosted.org/packages/fd/d6/67748211d119f3b6540baf90f92fae73ae51d5217b171b0e8b5f7e5d558f/pillow-12.0.0-pp311-pypy311_pp73-macosx_11_0_arm64.whl", hash = "sha256:805ebf596939e48dbb2e4922a1d3852cfc25c38160751ce02da93058b48d252a", size = 4614994, upload-time = "2025-10-15T18:24:01.669Z" },
 882      { url = "https://files.pythonhosted.org/packages/2d/e1/f8281e5d844c41872b273b9f2c34a4bf64ca08905668c8ae730eedc7c9fa/pillow-12.0.0-pp311-pypy311_pp73-manylinux2014_aarch64.manylinux_2_17_aarch64.whl", hash = "sha256:cae81479f77420d217def5f54b5b9d279804d17e982e0f2fa19b1d1e14ab5197", size = 5246639, upload-time = "2025-10-15T18:24:03.403Z" },
 883      { url = "https://files.pythonhosted.org/packages/94/5a/0d8ab8ffe8a102ff5df60d0de5af309015163bf710c7bb3e8311dd3b3ad0/pillow-12.0.0-pp311-pypy311_pp73-manylinux2014_x86_64.manylinux_2_17_x86_64.whl", hash = "sha256:aeaefa96c768fc66818730b952a862235d68825c178f1b3ffd4efd7ad2edcb7c", size = 6986839, upload-time = "2025-10-15T18:24:05.344Z" },
 884      { url = "https://files.pythonhosted.org/packages/20/2e/3434380e8110b76cd9eb00a363c484b050f949b4bbe84ba770bb8508a02c/pillow-12.0.0-pp311-pypy311_pp73-manylinux_2_27_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:09f2d0abef9e4e2f349305a4f8cc784a8a6c2f58a8c4892eea13b10a943bd26e", size = 5313505, upload-time = "2025-10-15T18:24:07.137Z" },
 885      { url = "https://files.pythonhosted.org/packages/57/ca/5a9d38900d9d74785141d6580950fe705de68af735ff6e727cb911b64740/pillow-12.0.0-pp311-pypy311_pp73-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:bdee52571a343d721fb2eb3b090a82d959ff37fc631e3f70422e0c2e029f3e76", size = 5963654, upload-time = "2025-10-15T18:24:09.579Z" },
 886      { url = "https://files.pythonhosted.org/packages/95/7e/f896623c3c635a90537ac093c6a618ebe1a90d87206e42309cb5d98a1b9e/pillow-12.0.0-pp311-pypy311_pp73-win_amd64.whl", hash = "sha256:b290fd8aa38422444d4b50d579de197557f182ef1068b75f5aa8558638b8d0a5", size = 6997850, upload-time = "2025-10-15T18:24:11.495Z" },
 887  ]
 888  
 889  [[package]]
 890  name = "pluggy"
 891  version = "1.5.0"
 892  source = { registry = "https://pypi.org/simple" }
 893  resolution-markers = [
 894      "python_full_version < '3.9'",
 895  ]
 896  sdist = { url = "https://files.pythonhosted.org/packages/96/2d/02d4312c973c6050a18b314a5ad0b3210edb65a906f868e31c111dede4a6/pluggy-1.5.0.tar.gz", hash = "sha256:2cffa88e94fdc978c4c574f15f9e59b7f4201d439195c3715ca9e2486f1d0cf1", size = 67955, upload-time = "2024-04-20T21:34:42.531Z" }
 897  wheels = [
 898      { url = "https://files.pythonhosted.org/packages/88/5f/e351af9a41f866ac3f1fac4ca0613908d9a41741cfcf2228f4ad853b697d/pluggy-1.5.0-py3-none-any.whl", hash = "sha256:44e1ad92c8ca002de6377e165f3e0f1be63266ab4d554740532335b9d75ea669", size = 20556, upload-time = "2024-04-20T21:34:40.434Z" },
 899  ]
 900  
 901  [[package]]
 902  name = "pluggy"
 903  version = "1.6.0"
 904  source = { registry = "https://pypi.org/simple" }
 905  resolution-markers = [
 906      "python_full_version >= '3.10'",
 907      "python_full_version == '3.9.*'",
 908  ]
 909  sdist = { url = "https://files.pythonhosted.org/packages/f9/e2/3e91f31a7d2b083fe6ef3fa267035b518369d9511ffab804f839851d2779/pluggy-1.6.0.tar.gz", hash = "sha256:7dcc130b76258d33b90f61b658791dede3486c3e6bfb003ee5c9bfb396dd22f3", size = 69412, upload-time = "2025-05-15T12:30:07.975Z" }
 910  wheels = [
 911      { url = "https://files.pythonhosted.org/packages/54/20/4d324d65cc6d9205fabedc306948156824eb9f0ee1633355a8f7ec5c66bf/pluggy-1.6.0-py3-none-any.whl", hash = "sha256:e920276dd6813095e9377c0bc5566d94c932c33b27a3e3945d8389c374dd4746", size = 20538, upload-time = "2025-05-15T12:30:06.134Z" },
 912  ]
 913  
 914  [[package]]
 915  name = "plumbum"
 916  version = "1.9.0"
 917  source = { registry = "https://pypi.org/simple" }
 918  resolution-markers = [
 919      "python_full_version < '3.9'",
 920  ]
 921  dependencies = [
 922      { name = "importlib-resources", marker = "python_full_version < '3.9'" },
 923      { name = "pywin32", marker = "python_full_version < '3.9' and platform_python_implementation != 'PyPy' and sys_platform == 'win32'" },
 924  ]
 925  sdist = { url = "https://files.pythonhosted.org/packages/f0/5d/49ba324ad4ae5b1a4caefafbce7a1648540129344481f2ed4ef6bb68d451/plumbum-1.9.0.tar.gz", hash = "sha256:e640062b72642c3873bd5bdc3effed75ba4d3c70ef6b6a7b907357a84d909219", size = 319083, upload-time = "2024-10-05T05:59:27.059Z" }
 926  wheels = [
 927      { url = "https://files.pythonhosted.org/packages/4f/9d/d03542c93bb3d448406731b80f39c3d5601282f778328c22c77d270f4ed4/plumbum-1.9.0-py3-none-any.whl", hash = "sha256:9fd0d3b0e8d86e4b581af36edf3f3bbe9d1ae15b45b8caab28de1bcb27aaa7f5", size = 127970, upload-time = "2024-10-05T05:59:25.102Z" },
 928  ]
 929  
 930  [[package]]
 931  name = "plumbum"
 932  version = "1.10.0"
 933  source = { registry = "https://pypi.org/simple" }
 934  resolution-markers = [
 935      "python_full_version >= '3.10'",
 936      "python_full_version == '3.9.*'",
 937  ]
 938  dependencies = [
 939      { name = "pywin32", marker = "python_full_version >= '3.9' and platform_python_implementation != 'PyPy' and sys_platform == 'win32'" },
 940  ]
 941  sdist = { url = "https://files.pythonhosted.org/packages/dc/c8/11a5f792704b70f071a3dbc329105a98e9cc8d25daaf09f733c44eb0ef8e/plumbum-1.10.0.tar.gz", hash = "sha256:f8cbf0ecec0b73ff4e349398b65112a9e3f9300e7dc019001217dcc148d5c97c", size = 320039, upload-time = "2025-10-31T05:02:48.697Z" }
 942  wheels = [
 943      { url = "https://files.pythonhosted.org/packages/79/ad/45312df6b63ba64ea35b8d8f5f0c577aac16e6b416eafe8e1cb34e03f9a7/plumbum-1.10.0-py3-none-any.whl", hash = "sha256:9583d737ac901c474d99d030e4d5eec4c4e6d2d7417b1cf49728cf3be34f6dc8", size = 127383, upload-time = "2025-10-31T05:02:47.002Z" },
 944  ]
 945  
 946  [[package]]
 947  name = "ply"
 948  version = "3.11"
 949  source = { registry = "https://pypi.org/simple" }
 950  sdist = { url = "https://files.pythonhosted.org/packages/e5/69/882ee5c9d017149285cab114ebeab373308ef0f874fcdac9beb90e0ac4da/ply-3.11.tar.gz", hash = "sha256:00c7c1aaa88358b9c765b6d3000c6eec0ba42abca5351b095321aef446081da3", size = 159130, upload-time = "2018-02-15T19:01:31.097Z" }
 951  wheels = [
 952      { url = "https://files.pythonhosted.org/packages/a3/58/35da89ee790598a0700ea49b2a66594140f44dec458c07e8e3d4979137fc/ply-3.11-py2.py3-none-any.whl", hash = "sha256:096f9b8350b65ebd2fd1346b12452efe5b9607f7482813ffca50c22722a807ce", size = 49567, upload-time = "2018-02-15T19:01:27.172Z" },
 953  ]
 954  
 955  [[package]]
 956  name = "pycparser"
 957  version = "2.23"
 958  source = { registry = "https://pypi.org/simple" }
 959  sdist = { url = "https://files.pythonhosted.org/packages/fe/cf/d2d3b9f5699fb1e4615c8e32ff220203e43b248e1dfcc6736ad9057731ca/pycparser-2.23.tar.gz", hash = "sha256:78816d4f24add8f10a06d6f05b4d424ad9e96cfebf68a4ddc99c65c0720d00c2", size = 173734, upload-time = "2025-09-09T13:23:47.91Z" }
 960  wheels = [
 961      { url = "https://files.pythonhosted.org/packages/a0/e3/59cd50310fc9b59512193629e1984c1f95e5c8ae6e5d8c69532ccc65a7fe/pycparser-2.23-py3-none-any.whl", hash = "sha256:e5c6e8d3fbad53479cab09ac03729e0a9faf2bee3db8208a550daf5af81a5934", size = 118140, upload-time = "2025-09-09T13:23:46.651Z" },
 962  ]
 963  
 964  [[package]]
 965  name = "pygments"
 966  version = "2.19.2"
 967  source = { registry = "https://pypi.org/simple" }
 968  sdist = { url = "https://files.pythonhosted.org/packages/b0/77/a5b8c569bf593b0140bde72ea885a803b82086995367bf2037de0159d924/pygments-2.19.2.tar.gz", hash = "sha256:636cb2477cec7f8952536970bc533bc43743542f70392ae026374600add5b887", size = 4968631, upload-time = "2025-06-21T13:39:12.283Z" }
 969  wheels = [
 970      { url = "https://files.pythonhosted.org/packages/c7/21/705964c7812476f378728bdf590ca4b771ec72385c533964653c68e86bdc/pygments-2.19.2-py3-none-any.whl", hash = "sha256:86540386c03d588bb81d44bc3928634ff26449851e99741617ecb9037ee5ec0b", size = 1225217, upload-time = "2025-06-21T13:39:07.939Z" },
 971  ]
 972  
 973  [[package]]
 974  name = "pypdfium2"
 975  version = "5.0.0"
 976  source = { registry = "https://pypi.org/simple" }
 977  sdist = { url = "https://files.pythonhosted.org/packages/fc/a1/34ebc27160533f4f11c4f2e36e4d0c3bc6fbef24b63b4582a376bbf26646/pypdfium2-5.0.0.tar.gz", hash = "sha256:666f66e8170f5502feac3b31c5c05a3697989c10e65e1a8503bf8dff8936b125", size = 243319, upload-time = "2025-10-26T13:31:41.987Z" }
 978  wheels = [
 979      { url = "https://files.pythonhosted.org/packages/13/bf/4259b23a88b92bec8199e1a08a0821dbfbb465629c203bdbc49e2f993940/pypdfium2-5.0.0-py3-none-macosx_11_0_arm64.whl", hash = "sha256:c477d68a0f32a22d6477d9aa9c5c2afae6512af1d5455a9ea561a224908f16ae", size = 2813187, upload-time = "2025-10-26T13:31:19.499Z" },
 980      { url = "https://files.pythonhosted.org/packages/48/5b/358ae0340300564b7d878cde62a40c01535ff1568393bdd5a8250278cfa9/pypdfium2-5.0.0-py3-none-macosx_11_0_x86_64.whl", hash = "sha256:753954aeb8e130507cb3b408da68f66a25c4b7e510bdfaf5458975ab8c8285c4", size = 2935797, upload-time = "2025-10-26T13:31:22.112Z" },
 981      { url = "https://files.pythonhosted.org/packages/af/74/94a4dc2f6891008111a9666214b5ef53a8390e3a324e957fdd93a8f18957/pypdfium2-5.0.0-py3-none-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:b1e50b08bde1c6c93685022ac72746ff099a7543f178d35e0a834f7e36bf401d", size = 2975686, upload-time = "2025-10-26T13:31:23.896Z" },
 982      { url = "https://files.pythonhosted.org/packages/b7/82/ce53918809fdc65d16b054e4d6e4f825b4e6513bcd67cfe89c13061c5ac5/pypdfium2-5.0.0-py3-none-manylinux_2_17_armv7l.manylinux2014_armv7l.whl", hash = "sha256:baf715937b3bc78312c2d07ab2b06684f57156adadc8e849f5892724f892648e", size = 2761052, upload-time = "2025-10-26T13:31:25.739Z" },
 983      { url = "https://files.pythonhosted.org/packages/6e/7b/b22dccb7ebd62b20bff1e8c3b06900bd1e529527326ddd9ee3c5157fbc6c/pypdfium2-5.0.0-py3-none-manylinux_2_17_i686.manylinux2014_i686.whl", hash = "sha256:f216423de641187c4e322992f3a97afc5ffa63b72d4ad30f8189cfa783c9d781", size = 3061679, upload-time = "2025-10-26T13:31:27.625Z" },
 984      { url = "https://files.pythonhosted.org/packages/01/ed/e0cbbf7430d908108e135bd9fff8195876b3ed7402fbe2893b09e9f53b88/pypdfium2-5.0.0-py3-none-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:4445d83ae3c6688667feba568b7b390b948c4a06ab94e576ad3b029b5567b44c", size = 2990851, upload-time = "2025-10-26T13:31:29.09Z" },
 985      { url = "https://files.pythonhosted.org/packages/48/5c/41595b3051b43d270fa249c7c0dec5cd52aa633ec64e5f9e1526692eef9d/pypdfium2-5.0.0-py3-none-musllinux_1_2_aarch64.whl", hash = "sha256:3b1cbc217a6accfab005806b53e467044f83fe61133df01a4fde94334e4655ac", size = 6320499, upload-time = "2025-10-26T13:31:31.335Z" },
 986      { url = "https://files.pythonhosted.org/packages/35/e2/7bfcdfd446fc3b086faca38621dc98dd6feafa9c1b2102a59b3a68862e03/pypdfium2-5.0.0-py3-none-musllinux_1_2_i686.whl", hash = "sha256:2f050cca56c4d85c24dcb572344cf5e54ebcd0a0dd351fcf6b5117e72474382c", size = 6329280, upload-time = "2025-10-26T13:31:33.421Z" },
 987      { url = "https://files.pythonhosted.org/packages/3d/6a/626f358ecd363afd3306bd15e98a040d6b2f4db9482ca7827dcf34677994/pypdfium2-5.0.0-py3-none-musllinux_1_2_x86_64.whl", hash = "sha256:4ee80e08a5c93a8e0f9e26a1978d4e0a31f0122a33351c260a6e436300d95075", size = 6408895, upload-time = "2025-10-26T13:31:35.055Z" },
 988      { url = "https://files.pythonhosted.org/packages/cc/87/79b9aa6d7f58959c821fb3d6e679ad288d17773c5ef59c69889bb1d3af53/pypdfium2-5.0.0-py3-none-win32.whl", hash = "sha256:aafb55d57f03c8cf482557ed421d40aed943cd563628a3df8515f301725f8e49", size = 2986180, upload-time = "2025-10-26T13:31:36.633Z" },
 989      { url = "https://files.pythonhosted.org/packages/21/46/21de463f575a85dc8973fdf89f7a103d09da553e896161536d7cc73950fd/pypdfium2-5.0.0-py3-none-win_amd64.whl", hash = "sha256:de2201d4e9e423779d2e3b2c2368591d6826153a009146eaa105b501a213b299", size = 3094011, upload-time = "2025-10-26T13:31:38.341Z" },
 990      { url = "https://files.pythonhosted.org/packages/ae/43/2b0607ef7f16d63fbe00de728151a090397ef5b3b9147b4aefe975d17106/pypdfium2-5.0.0-py3-none-win_arm64.whl", hash = "sha256:0a2a473fe95802e7a5f4140f25e5cd036cf17f060f27ee2d28c3977206add763", size = 2939015, upload-time = "2025-10-26T13:31:40.531Z" },
 991  ]
 992  
 993  [[package]]
 994  name = "pytest"
 995  version = "8.3.5"
 996  source = { registry = "https://pypi.org/simple" }
 997  resolution-markers = [
 998      "python_full_version < '3.9'",
 999  ]
1000  dependencies = [
1001      { name = "colorama", marker = "python_full_version < '3.9' and sys_platform == 'win32'" },
1002      { name = "exceptiongroup", marker = "python_full_version < '3.9'" },
1003      { name = "iniconfig", version = "2.1.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
1004      { name = "packaging", marker = "python_full_version < '3.9'" },
1005      { name = "pluggy", version = "1.5.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version < '3.9'" },
1006      { name = "tomli", marker = "python_full_version < '3.9'" },
1007  ]
1008  sdist = { url = "https://files.pythonhosted.org/packages/ae/3c/c9d525a414d506893f0cd8a8d0de7706446213181570cdbd766691164e40/pytest-8.3.5.tar.gz", hash = "sha256:f4efe70cc14e511565ac476b57c279e12a855b11f48f212af1080ef2263d3845", size = 1450891, upload-time = "2025-03-02T12:54:54.503Z" }
1009  wheels = [
1010      { url = "https://files.pythonhosted.org/packages/30/3d/64ad57c803f1fa1e963a7946b6e0fea4a70df53c1a7fed304586539c2bac/pytest-8.3.5-py3-none-any.whl", hash = "sha256:c69214aa47deac29fad6c2a4f590b9c4a9fdb16a403176fe154b79c0b4d4d820", size = 343634, upload-time = "2025-03-02T12:54:52.069Z" },
1011  ]
1012  
1013  [[package]]
1014  name = "pytest"
1015  version = "8.4.2"
1016  source = { registry = "https://pypi.org/simple" }
1017  resolution-markers = [
1018      "python_full_version == '3.9.*'",
1019  ]
1020  dependencies = [
1021      { name = "colorama", marker = "python_full_version == '3.9.*' and sys_platform == 'win32'" },
1022      { name = "exceptiongroup", marker = "python_full_version == '3.9.*'" },
1023      { name = "iniconfig", version = "2.1.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version == '3.9.*'" },
1024      { name = "packaging", marker = "python_full_version == '3.9.*'" },
1025      { name = "pluggy", version = "1.6.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version == '3.9.*'" },
1026      { name = "pygments", marker = "python_full_version == '3.9.*'" },
1027      { name = "tomli", marker = "python_full_version == '3.9.*'" },
1028  ]
1029  sdist = { url = "https://files.pythonhosted.org/packages/a3/5c/00a0e072241553e1a7496d638deababa67c5058571567b92a7eaa258397c/pytest-8.4.2.tar.gz", hash = "sha256:86c0d0b93306b961d58d62a4db4879f27fe25513d4b969df351abdddb3c30e01", size = 1519618, upload-time = "2025-09-04T14:34:22.711Z" }
1030  wheels = [
1031      { url = "https://files.pythonhosted.org/packages/a8/a4/20da314d277121d6534b3a980b29035dcd51e6744bd79075a6ce8fa4eb8d/pytest-8.4.2-py3-none-any.whl", hash = "sha256:872f880de3fc3a5bdc88a11b39c9710c3497a547cfa9320bc3c5e62fbf272e79", size = 365750, upload-time = "2025-09-04T14:34:20.226Z" },
1032  ]
1033  
1034  [[package]]
1035  name = "pytest"
1036  version = "9.0.1"
1037  source = { registry = "https://pypi.org/simple" }
1038  resolution-markers = [
1039      "python_full_version >= '3.10'",
1040  ]
1041  dependencies = [
1042      { name = "colorama", marker = "python_full_version >= '3.10' and sys_platform == 'win32'" },
1043      { name = "exceptiongroup", marker = "python_full_version == '3.10.*'" },
1044      { name = "iniconfig", version = "2.3.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.10'" },
1045      { name = "packaging", marker = "python_full_version >= '3.10'" },
1046      { name = "pluggy", version = "1.6.0", source = { registry = "https://pypi.org/simple" }, marker = "python_full_version >= '3.10'" },
1047      { name = "pygments", marker = "python_full_version >= '3.10'" },
1048      { name = "tomli", marker = "python_full_version == '3.10.*'" },
1049  ]
1050  sdist = { url = "https://files.pythonhosted.org/packages/07/56/f013048ac4bc4c1d9be45afd4ab209ea62822fb1598f40687e6bf45dcea4/pytest-9.0.1.tar.gz", hash = "sha256:3e9c069ea73583e255c3b21cf46b8d3c56f6e3a1a8f6da94ccb0fcf57b9d73c8", size = 1564125, upload-time = "2025-11-12T13:05:09.333Z" }
1051  wheels = [
1052      { url = "https://files.pythonhosted.org/packages/0b/8b/6300fb80f858cda1c51ffa17075df5d846757081d11ab4aa35cef9e6258b/pytest-9.0.1-py3-none-any.whl", hash = "sha256:67be0030d194df2dfa7b556f2e56fb3c3315bd5c8822c6951162b92b32ce7dad", size = 373668, upload-time = "2025-11-12T13:05:07.379Z" },
1053  ]
1054  
1055  [[package]]
1056  name = "pywin32"
1057  version = "311"
1058  source = { registry = "https://pypi.org/simple" }
1059  wheels = [
1060      { url = "https://files.pythonhosted.org/packages/7b/40/44efbb0dfbd33aca6a6483191dae0716070ed99e2ecb0c53683f400a0b4f/pywin32-311-cp310-cp310-win32.whl", hash = "sha256:d03ff496d2a0cd4a5893504789d4a15399133fe82517455e78bad62efbb7f0a3", size = 8760432, upload-time = "2025-07-14T20:13:05.9Z" },
1061      { url = "https://files.pythonhosted.org/packages/5e/bf/360243b1e953bd254a82f12653974be395ba880e7ec23e3731d9f73921cc/pywin32-311-cp310-cp310-win_amd64.whl", hash = "sha256:797c2772017851984b97180b0bebe4b620bb86328e8a884bb626156295a63b3b", size = 9590103, upload-time = "2025-07-14T20:13:07.698Z" },
1062      { url = "https://files.pythonhosted.org/packages/57/38/d290720e6f138086fb3d5ffe0b6caa019a791dd57866940c82e4eeaf2012/pywin32-311-cp310-cp310-win_arm64.whl", hash = "sha256:0502d1facf1fed4839a9a51ccbcc63d952cf318f78ffc00a7e78528ac27d7a2b", size = 8778557, upload-time = "2025-07-14T20:13:11.11Z" },
1063      { url = "https://files.pythonhosted.org/packages/7c/af/449a6a91e5d6db51420875c54f6aff7c97a86a3b13a0b4f1a5c13b988de3/pywin32-311-cp311-cp311-win32.whl", hash = "sha256:184eb5e436dea364dcd3d2316d577d625c0351bf237c4e9a5fabbcfa5a58b151", size = 8697031, upload-time = "2025-07-14T20:13:13.266Z" },
1064      { url = "https://files.pythonhosted.org/packages/51/8f/9bb81dd5bb77d22243d33c8397f09377056d5c687aa6d4042bea7fbf8364/pywin32-311-cp311-cp311-win_amd64.whl", hash = "sha256:3ce80b34b22b17ccbd937a6e78e7225d80c52f5ab9940fe0506a1a16f3dab503", size = 9508308, upload-time = "2025-07-14T20:13:15.147Z" },
1065      { url = "https://files.pythonhosted.org/packages/44/7b/9c2ab54f74a138c491aba1b1cd0795ba61f144c711daea84a88b63dc0f6c/pywin32-311-cp311-cp311-win_arm64.whl", hash = "sha256:a733f1388e1a842abb67ffa8e7aad0e70ac519e09b0f6a784e65a136ec7cefd2", size = 8703930, upload-time = "2025-07-14T20:13:16.945Z" },
1066      { url = "https://files.pythonhosted.org/packages/e7/ab/01ea1943d4eba0f850c3c61e78e8dd59757ff815ff3ccd0a84de5f541f42/pywin32-311-cp312-cp312-win32.whl", hash = "sha256:750ec6e621af2b948540032557b10a2d43b0cee2ae9758c54154d711cc852d31", size = 8706543, upload-time = "2025-07-14T20:13:20.765Z" },
1067      { url = "https://files.pythonhosted.org/packages/d1/a8/a0e8d07d4d051ec7502cd58b291ec98dcc0c3fff027caad0470b72cfcc2f/pywin32-311-cp312-cp312-win_amd64.whl", hash = "sha256:b8c095edad5c211ff31c05223658e71bf7116daa0ecf3ad85f3201ea3190d067", size = 9495040, upload-time = "2025-07-14T20:13:22.543Z" },
1068      { url = "https://files.pythonhosted.org/packages/ba/3a/2ae996277b4b50f17d61f0603efd8253cb2d79cc7ae159468007b586396d/pywin32-311-cp312-cp312-win_arm64.whl", hash = "sha256:e286f46a9a39c4a18b319c28f59b61de793654af2f395c102b4f819e584b5852", size = 8710102, upload-time = "2025-07-14T20:13:24.682Z" },
1069      { url = "https://files.pythonhosted.org/packages/a5/be/3fd5de0979fcb3994bfee0d65ed8ca9506a8a1260651b86174f6a86f52b3/pywin32-311-cp313-cp313-win32.whl", hash = "sha256:f95ba5a847cba10dd8c4d8fefa9f2a6cf283b8b88ed6178fa8a6c1ab16054d0d", size = 8705700, upload-time = "2025-07-14T20:13:26.471Z" },
1070      { url = "https://files.pythonhosted.org/packages/e3/28/e0a1909523c6890208295a29e05c2adb2126364e289826c0a8bc7297bd5c/pywin32-311-cp313-cp313-win_amd64.whl", hash = "sha256:718a38f7e5b058e76aee1c56ddd06908116d35147e133427e59a3983f703a20d", size = 9494700, upload-time = "2025-07-14T20:13:28.243Z" },
1071      { url = "https://files.pythonhosted.org/packages/04/bf/90339ac0f55726dce7d794e6d79a18a91265bdf3aa70b6b9ca52f35e022a/pywin32-311-cp313-cp313-win_arm64.whl", hash = "sha256:7b4075d959648406202d92a2310cb990fea19b535c7f4a78d3f5e10b926eeb8a", size = 8709318, upload-time = "2025-07-14T20:13:30.348Z" },
1072      { url = "https://files.pythonhosted.org/packages/c9/31/097f2e132c4f16d99a22bfb777e0fd88bd8e1c634304e102f313af69ace5/pywin32-311-cp314-cp314-win32.whl", hash = "sha256:b7a2c10b93f8986666d0c803ee19b5990885872a7de910fc460f9b0c2fbf92ee", size = 8840714, upload-time = "2025-07-14T20:13:32.449Z" },
1073      { url = "https://files.pythonhosted.org/packages/90/4b/07c77d8ba0e01349358082713400435347df8426208171ce297da32c313d/pywin32-311-cp314-cp314-win_amd64.whl", hash = "sha256:3aca44c046bd2ed8c90de9cb8427f581c479e594e99b5c0bb19b29c10fd6cb87", size = 9656800, upload-time = "2025-07-14T20:13:34.312Z" },
1074      { url = "https://files.pythonhosted.org/packages/c0/d2/21af5c535501a7233e734b8af901574572da66fcc254cb35d0609c9080dd/pywin32-311-cp314-cp314-win_arm64.whl", hash = "sha256:a508e2d9025764a8270f93111a970e1d0fbfc33f4153b388bb649b7eec4f9b42", size = 8932540, upload-time = "2025-07-14T20:13:36.379Z" },
1075      { url = "https://files.pythonhosted.org/packages/75/20/6cd04d636a4c83458ecbb7c8220c13786a1a80d3f5fb568df39310e73e98/pywin32-311-cp38-cp38-win32.whl", hash = "sha256:6c6f2969607b5023b0d9ce2541f8d2cbb01c4f46bc87456017cf63b73f1e2d8c", size = 8766775, upload-time = "2025-07-14T20:12:55.029Z" },
1076      { url = "https://files.pythonhosted.org/packages/ff/6c/94c10268bae5d0d0c6509bdfb5aa08882d11a9ccdf89ff1cde59a6161afb/pywin32-311-cp38-cp38-win_amd64.whl", hash = "sha256:c8015b09fb9a5e188f83b7b04de91ddca4658cee2ae6f3bc483f0b21a77ef6cd", size = 9594743, upload-time = "2025-07-14T20:12:57.59Z" },
1077      { url = "https://files.pythonhosted.org/packages/59/42/b86689aac0cdaee7ae1c58d464b0ff04ca909c19bb6502d4973cdd9f9544/pywin32-311-cp39-cp39-win32.whl", hash = "sha256:aba8f82d551a942cb20d4a83413ccbac30790b50efb89a75e4f586ac0bb8056b", size = 8760837, upload-time = "2025-07-14T20:12:59.59Z" },
1078      { url = "https://files.pythonhosted.org/packages/9f/8a/1403d0353f8c5a2f0829d2b1c4becbf9da2f0a4d040886404fc4a5431e4d/pywin32-311-cp39-cp39-win_amd64.whl", hash = "sha256:e0c4cfb0621281fe40387df582097fd796e80430597cb9944f0ae70447bacd91", size = 9590187, upload-time = "2025-07-14T20:13:01.419Z" },
1079      { url = "https://files.pythonhosted.org/packages/60/22/e0e8d802f124772cec9c75430b01a212f86f9de7546bda715e54140d5aeb/pywin32-311-cp39-cp39-win_arm64.whl", hash = "sha256:62ea666235135fee79bb154e695f3ff67370afefd71bd7fea7512fc70ef31e3d", size = 8778162, upload-time = "2025-07-14T20:13:03.544Z" },
1080  ]
1081  
1082  [[package]]
1083  name = "tomli"
1084  version = "2.3.0"
1085  source = { registry = "https://pypi.org/simple" }
1086  sdist = { url = "https://files.pythonhosted.org/packages/52/ed/3f73f72945444548f33eba9a87fc7a6e969915e7b1acc8260b30e1f76a2f/tomli-2.3.0.tar.gz", hash = "sha256:64be704a875d2a59753d80ee8a533c3fe183e3f06807ff7dc2232938ccb01549", size = 17392, upload-time = "2025-10-08T22:01:47.119Z" }
1087  wheels = [
1088      { url = "https://files.pythonhosted.org/packages/b3/2e/299f62b401438d5fe1624119c723f5d877acc86a4c2492da405626665f12/tomli-2.3.0-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:88bd15eb972f3664f5ed4b57c1634a97153b4bac4479dcb6a495f41921eb7f45", size = 153236, upload-time = "2025-10-08T22:01:00.137Z" },
1089      { url = "https://files.pythonhosted.org/packages/86/7f/d8fffe6a7aefdb61bced88fcb5e280cfd71e08939da5894161bd71bea022/tomli-2.3.0-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:883b1c0d6398a6a9d29b508c331fa56adbcdff647f6ace4dfca0f50e90dfd0ba", size = 148084, upload-time = "2025-10-08T22:01:01.63Z" },
1090      { url = "https://files.pythonhosted.org/packages/47/5c/24935fb6a2ee63e86d80e4d3b58b222dafaf438c416752c8b58537c8b89a/tomli-2.3.0-cp311-cp311-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:d1381caf13ab9f300e30dd8feadb3de072aeb86f1d34a8569453ff32a7dea4bf", size = 234832, upload-time = "2025-10-08T22:01:02.543Z" },
1091      { url = "https://files.pythonhosted.org/packages/89/da/75dfd804fc11e6612846758a23f13271b76d577e299592b4371a4ca4cd09/tomli-2.3.0-cp311-cp311-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:a0e285d2649b78c0d9027570d4da3425bdb49830a6156121360b3f8511ea3441", size = 242052, upload-time = "2025-10-08T22:01:03.836Z" },
1092      { url = "https://files.pythonhosted.org/packages/70/8c/f48ac899f7b3ca7eb13af73bacbc93aec37f9c954df3c08ad96991c8c373/tomli-2.3.0-cp311-cp311-musllinux_1_2_aarch64.whl", hash = "sha256:0a154a9ae14bfcf5d8917a59b51ffd5a3ac1fd149b71b47a3a104ca4edcfa845", size = 239555, upload-time = "2025-10-08T22:01:04.834Z" },
1093      { url = "https://files.pythonhosted.org/packages/ba/28/72f8afd73f1d0e7829bfc093f4cb98ce0a40ffc0cc997009ee1ed94ba705/tomli-2.3.0-cp311-cp311-musllinux_1_2_x86_64.whl", hash = "sha256:74bf8464ff93e413514fefd2be591c3b0b23231a77f901db1eb30d6f712fc42c", size = 245128, upload-time = "2025-10-08T22:01:05.84Z" },
1094      { url = "https://files.pythonhosted.org/packages/b6/eb/a7679c8ac85208706d27436e8d421dfa39d4c914dcf5fa8083a9305f58d9/tomli-2.3.0-cp311-cp311-win32.whl", hash = "sha256:00b5f5d95bbfc7d12f91ad8c593a1659b6387b43f054104cda404be6bda62456", size = 96445, upload-time = "2025-10-08T22:01:06.896Z" },
1095      { url = "https://files.pythonhosted.org/packages/0a/fe/3d3420c4cb1ad9cb462fb52967080575f15898da97e21cb6f1361d505383/tomli-2.3.0-cp311-cp311-win_amd64.whl", hash = "sha256:4dc4ce8483a5d429ab602f111a93a6ab1ed425eae3122032db7e9acf449451be", size = 107165, upload-time = "2025-10-08T22:01:08.107Z" },
1096      { url = "https://files.pythonhosted.org/packages/ff/b7/40f36368fcabc518bb11c8f06379a0fd631985046c038aca08c6d6a43c6e/tomli-2.3.0-cp312-cp312-macosx_10_13_x86_64.whl", hash = "sha256:d7d86942e56ded512a594786a5ba0a5e521d02529b3826e7761a05138341a2ac", size = 154891, upload-time = "2025-10-08T22:01:09.082Z" },
1097      { url = "https://files.pythonhosted.org/packages/f9/3f/d9dd692199e3b3aab2e4e4dd948abd0f790d9ded8cd10cbaae276a898434/tomli-2.3.0-cp312-cp312-macosx_11_0_arm64.whl", hash = "sha256:73ee0b47d4dad1c5e996e3cd33b8a76a50167ae5f96a2607cbe8cc773506ab22", size = 148796, upload-time = "2025-10-08T22:01:10.266Z" },
1098      { url = "https://files.pythonhosted.org/packages/60/83/59bff4996c2cf9f9387a0f5a3394629c7efa5ef16142076a23a90f1955fa/tomli-2.3.0-cp312-cp312-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:792262b94d5d0a466afb5bc63c7daa9d75520110971ee269152083270998316f", size = 242121, upload-time = "2025-10-08T22:01:11.332Z" },
1099      { url = "https://files.pythonhosted.org/packages/45/e5/7c5119ff39de8693d6baab6c0b6dcb556d192c165596e9fc231ea1052041/tomli-2.3.0-cp312-cp312-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:4f195fe57ecceac95a66a75ac24d9d5fbc98ef0962e09b2eddec5d39375aae52", size = 250070, upload-time = "2025-10-08T22:01:12.498Z" },
1100      { url = "https://files.pythonhosted.org/packages/45/12/ad5126d3a278f27e6701abde51d342aa78d06e27ce2bb596a01f7709a5a2/tomli-2.3.0-cp312-cp312-musllinux_1_2_aarch64.whl", hash = "sha256:e31d432427dcbf4d86958c184b9bfd1e96b5b71f8eb17e6d02531f434fd335b8", size = 245859, upload-time = "2025-10-08T22:01:13.551Z" },
1101      { url = "https://files.pythonhosted.org/packages/fb/a1/4d6865da6a71c603cfe6ad0e6556c73c76548557a8d658f9e3b142df245f/tomli-2.3.0-cp312-cp312-musllinux_1_2_x86_64.whl", hash = "sha256:7b0882799624980785240ab732537fcfc372601015c00f7fc367c55308c186f6", size = 250296, upload-time = "2025-10-08T22:01:14.614Z" },
1102      { url = "https://files.pythonhosted.org/packages/a0/b7/a7a7042715d55c9ba6e8b196d65d2cb662578b4d8cd17d882d45322b0d78/tomli-2.3.0-cp312-cp312-win32.whl", hash = "sha256:ff72b71b5d10d22ecb084d345fc26f42b5143c5533db5e2eaba7d2d335358876", size = 97124, upload-time = "2025-10-08T22:01:15.629Z" },
1103      { url = "https://files.pythonhosted.org/packages/06/1e/f22f100db15a68b520664eb3328fb0ae4e90530887928558112c8d1f4515/tomli-2.3.0-cp312-cp312-win_amd64.whl", hash = "sha256:1cb4ed918939151a03f33d4242ccd0aa5f11b3547d0cf30f7c74a408a5b99878", size = 107698, upload-time = "2025-10-08T22:01:16.51Z" },
1104      { url = "https://files.pythonhosted.org/packages/89/48/06ee6eabe4fdd9ecd48bf488f4ac783844fd777f547b8d1b61c11939974e/tomli-2.3.0-cp313-cp313-macosx_10_13_x86_64.whl", hash = "sha256:5192f562738228945d7b13d4930baffda67b69425a7f0da96d360b0a3888136b", size = 154819, upload-time = "2025-10-08T22:01:17.964Z" },
1105      { url = "https://files.pythonhosted.org/packages/f1/01/88793757d54d8937015c75dcdfb673c65471945f6be98e6a0410fba167ed/tomli-2.3.0-cp313-cp313-macosx_11_0_arm64.whl", hash = "sha256:be71c93a63d738597996be9528f4abe628d1adf5e6eb11607bc8fe1a510b5dae", size = 148766, upload-time = "2025-10-08T22:01:18.959Z" },
1106      { url = "https://files.pythonhosted.org/packages/42/17/5e2c956f0144b812e7e107f94f1cc54af734eb17b5191c0bbfb72de5e93e/tomli-2.3.0-cp313-cp313-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:c4665508bcbac83a31ff8ab08f424b665200c0e1e645d2bd9ab3d3e557b6185b", size = 240771, upload-time = "2025-10-08T22:01:20.106Z" },
1107      { url = "https://files.pythonhosted.org/packages/d5/f4/0fbd014909748706c01d16824eadb0307115f9562a15cbb012cd9b3512c5/tomli-2.3.0-cp313-cp313-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:4021923f97266babc6ccab9f5068642a0095faa0a51a246a6a02fccbb3514eaf", size = 248586, upload-time = "2025-10-08T22:01:21.164Z" },
1108      { url = "https://files.pythonhosted.org/packages/30/77/fed85e114bde5e81ecf9bc5da0cc69f2914b38f4708c80ae67d0c10180c5/tomli-2.3.0-cp313-cp313-musllinux_1_2_aarch64.whl", hash = "sha256:a4ea38c40145a357d513bffad0ed869f13c1773716cf71ccaa83b0fa0cc4e42f", size = 244792, upload-time = "2025-10-08T22:01:22.417Z" },
1109      { url = "https://files.pythonhosted.org/packages/55/92/afed3d497f7c186dc71e6ee6d4fcb0acfa5f7d0a1a2878f8beae379ae0cc/tomli-2.3.0-cp313-cp313-musllinux_1_2_x86_64.whl", hash = "sha256:ad805ea85eda330dbad64c7ea7a4556259665bdf9d2672f5dccc740eb9d3ca05", size = 248909, upload-time = "2025-10-08T22:01:23.859Z" },
1110      { url = "https://files.pythonhosted.org/packages/f8/84/ef50c51b5a9472e7265ce1ffc7f24cd4023d289e109f669bdb1553f6a7c2/tomli-2.3.0-cp313-cp313-win32.whl", hash = "sha256:97d5eec30149fd3294270e889b4234023f2c69747e555a27bd708828353ab606", size = 96946, upload-time = "2025-10-08T22:01:24.893Z" },
1111      { url = "https://files.pythonhosted.org/packages/b2/b7/718cd1da0884f281f95ccfa3a6cc572d30053cba64603f79d431d3c9b61b/tomli-2.3.0-cp313-cp313-win_amd64.whl", hash = "sha256:0c95ca56fbe89e065c6ead5b593ee64b84a26fca063b5d71a1122bf26e533999", size = 107705, upload-time = "2025-10-08T22:01:26.153Z" },
1112      { url = "https://files.pythonhosted.org/packages/19/94/aeafa14a52e16163008060506fcb6aa1949d13548d13752171a755c65611/tomli-2.3.0-cp314-cp314-macosx_10_13_x86_64.whl", hash = "sha256:cebc6fe843e0733ee827a282aca4999b596241195f43b4cc371d64fc6639da9e", size = 154244, upload-time = "2025-10-08T22:01:27.06Z" },
1113      { url = "https://files.pythonhosted.org/packages/db/e4/1e58409aa78eefa47ccd19779fc6f36787edbe7d4cd330eeeedb33a4515b/tomli-2.3.0-cp314-cp314-macosx_11_0_arm64.whl", hash = "sha256:4c2ef0244c75aba9355561272009d934953817c49f47d768070c3c94355c2aa3", size = 148637, upload-time = "2025-10-08T22:01:28.059Z" },
1114      { url = "https://files.pythonhosted.org/packages/26/b6/d1eccb62f665e44359226811064596dd6a366ea1f985839c566cd61525ae/tomli-2.3.0-cp314-cp314-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:c22a8bf253bacc0cf11f35ad9808b6cb75ada2631c2d97c971122583b129afbc", size = 241925, upload-time = "2025-10-08T22:01:29.066Z" },
1115      { url = "https://files.pythonhosted.org/packages/70/91/7cdab9a03e6d3d2bb11beae108da5bdc1c34bdeb06e21163482544ddcc90/tomli-2.3.0-cp314-cp314-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:0eea8cc5c5e9f89c9b90c4896a8deefc74f518db5927d0e0e8d4a80953d774d0", size = 249045, upload-time = "2025-10-08T22:01:31.98Z" },
1116      { url = "https://files.pythonhosted.org/packages/15/1b/8c26874ed1f6e4f1fcfeb868db8a794cbe9f227299402db58cfcc858766c/tomli-2.3.0-cp314-cp314-musllinux_1_2_aarch64.whl", hash = "sha256:b74a0e59ec5d15127acdabd75ea17726ac4c5178ae51b85bfe39c4f8a278e879", size = 245835, upload-time = "2025-10-08T22:01:32.989Z" },
1117      { url = "https://files.pythonhosted.org/packages/fd/42/8e3c6a9a4b1a1360c1a2a39f0b972cef2cc9ebd56025168c4137192a9321/tomli-2.3.0-cp314-cp314-musllinux_1_2_x86_64.whl", hash = "sha256:b5870b50c9db823c595983571d1296a6ff3e1b88f734a4c8f6fc6188397de005", size = 253109, upload-time = "2025-10-08T22:01:34.052Z" },
1118      { url = "https://files.pythonhosted.org/packages/22/0c/b4da635000a71b5f80130937eeac12e686eefb376b8dee113b4a582bba42/tomli-2.3.0-cp314-cp314-win32.whl", hash = "sha256:feb0dacc61170ed7ab602d3d972a58f14ee3ee60494292d384649a3dc38ef463", size = 97930, upload-time = "2025-10-08T22:01:35.082Z" },
1119      { url = "https://files.pythonhosted.org/packages/b9/74/cb1abc870a418ae99cd5c9547d6bce30701a954e0e721821df483ef7223c/tomli-2.3.0-cp314-cp314-win_amd64.whl", hash = "sha256:b273fcbd7fc64dc3600c098e39136522650c49bca95df2d11cf3b626422392c8", size = 107964, upload-time = "2025-10-08T22:01:36.057Z" },
1120      { url = "https://files.pythonhosted.org/packages/54/78/5c46fff6432a712af9f792944f4fcd7067d8823157949f4e40c56b8b3c83/tomli-2.3.0-cp314-cp314t-macosx_10_13_x86_64.whl", hash = "sha256:940d56ee0410fa17ee1f12b817b37a4d4e4dc4d27340863cc67236c74f582e77", size = 163065, upload-time = "2025-10-08T22:01:37.27Z" },
1121      { url = "https://files.pythonhosted.org/packages/39/67/f85d9bd23182f45eca8939cd2bc7050e1f90c41f4a2ecbbd5963a1d1c486/tomli-2.3.0-cp314-cp314t-macosx_11_0_arm64.whl", hash = "sha256:f85209946d1fe94416debbb88d00eb92ce9cd5266775424ff81bc959e001acaf", size = 159088, upload-time = "2025-10-08T22:01:38.235Z" },
1122      { url = "https://files.pythonhosted.org/packages/26/5a/4b546a0405b9cc0659b399f12b6adb750757baf04250b148d3c5059fc4eb/tomli-2.3.0-cp314-cp314t-manylinux2014_aarch64.manylinux_2_17_aarch64.manylinux_2_28_aarch64.whl", hash = "sha256:a56212bdcce682e56b0aaf79e869ba5d15a6163f88d5451cbde388d48b13f530", size = 268193, upload-time = "2025-10-08T22:01:39.712Z" },
1123      { url = "https://files.pythonhosted.org/packages/42/4f/2c12a72ae22cf7b59a7fe75b3465b7aba40ea9145d026ba41cb382075b0e/tomli-2.3.0-cp314-cp314t-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl", hash = "sha256:c5f3ffd1e098dfc032d4d3af5c0ac64f6d286d98bc148698356847b80fa4de1b", size = 275488, upload-time = "2025-10-08T22:01:40.773Z" },
1124      { url = "https://files.pythonhosted.org/packages/92/04/a038d65dbe160c3aa5a624e93ad98111090f6804027d474ba9c37c8ae186/tomli-2.3.0-cp314-cp314t-musllinux_1_2_aarch64.whl", hash = "sha256:5e01decd096b1530d97d5d85cb4dff4af2d8347bd35686654a004f8dea20fc67", size = 272669, upload-time = "2025-10-08T22:01:41.824Z" },
1125      { url = "https://files.pythonhosted.org/packages/be/2f/8b7c60a9d1612a7cbc39ffcca4f21a73bf368a80fc25bccf8253e2563267/tomli-2.3.0-cp314-cp314t-musllinux_1_2_x86_64.whl", hash = "sha256:8a35dd0e643bb2610f156cca8db95d213a90015c11fee76c946aa62b7ae7e02f", size = 279709, upload-time = "2025-10-08T22:01:43.177Z" },
1126      { url = "https://files.pythonhosted.org/packages/7e/46/cc36c679f09f27ded940281c38607716c86cf8ba4a518d524e349c8b4874/tomli-2.3.0-cp314-cp314t-win32.whl", hash = "sha256:a1f7f282fe248311650081faafa5f4732bdbfef5d45fe3f2e702fbc6f2d496e0", size = 107563, upload-time = "2025-10-08T22:01:44.233Z" },
1127      { url = "https://files.pythonhosted.org/packages/84/ff/426ca8683cf7b753614480484f6437f568fd2fda2edbdf57a2d3d8b27a0b/tomli-2.3.0-cp314-cp314t-win_amd64.whl", hash = "sha256:70a251f8d4ba2d9ac2542eecf008b3c8a9fc5c3f9f02c56a9d7952612be2fdba", size = 119756, upload-time = "2025-10-08T22:01:45.234Z" },
1128      { url = "https://files.pythonhosted.org/packages/77/b8/0135fadc89e73be292b473cb820b4f5a08197779206b33191e801feeae40/tomli-2.3.0-py3-none-any.whl", hash = "sha256:e95b1af3c5b07d9e643909b5abbec77cd9f1217e6d0bca72b0234736b9fb1f1b", size = 14408, upload-time = "2025-10-08T22:01:46.04Z" },
1129  ]
1130  
1131  [[package]]
1132  name = "typing-extensions"
1133  version = "4.13.2"
1134  source = { registry = "https://pypi.org/simple" }
1135  resolution-markers = [
1136      "python_full_version < '3.9'",
1137  ]
1138  sdist = { url = "https://files.pythonhosted.org/packages/f6/37/23083fcd6e35492953e8d2aaaa68b860eb422b34627b13f2ce3eb6106061/typing_extensions-4.13.2.tar.gz", hash = "sha256:e6c81219bd689f51865d9e372991c540bda33a0379d5573cddb9a3a23f7caaef", size = 106967, upload-time = "2025-04-10T14:19:05.416Z" }
1139  wheels = [
1140      { url = "https://files.pythonhosted.org/packages/8b/54/b1ae86c0973cc6f0210b53d508ca3641fb6d0c56823f288d108bc7ab3cc8/typing_extensions-4.13.2-py3-none-any.whl", hash = "sha256:a439e7c04b49fec3e5d3e2beaa21755cadbbdc391694e28ccdd36ca4a1408f8c", size = 45806, upload-time = "2025-04-10T14:19:03.967Z" },
1141  ]
1142  
1143  [[package]]
1144  name = "typing-extensions"
1145  version = "4.15.0"
1146  source = { registry = "https://pypi.org/simple" }
1147  resolution-markers = [
1148      "python_full_version >= '3.10'",
1149      "python_full_version == '3.9.*'",
1150  ]
1151  sdist = { url = "https://files.pythonhosted.org/packages/72/94/1a15dd82efb362ac84269196e94cf00f187f7ed21c242792a923cdb1c61f/typing_extensions-4.15.0.tar.gz", hash = "sha256:0cea48d173cc12fa28ecabc3b837ea3cf6f38c6d1136f85cbaaf598984861466", size = 109391, upload-time = "2025-08-25T13:49:26.313Z" }
1152  wheels = [
1153      { url = "https://files.pythonhosted.org/packages/18/67/36e9267722cc04a6b9f15c7f3441c2363321a3ea07da7ae0c0707beb2a9c/typing_extensions-4.15.0-py3-none-any.whl", hash = "sha256:f0fa19c6845758ab08074a0cfa8b7aecb71c999ca73d62883bc25cc018c4e548", size = 44614, upload-time = "2025-08-25T13:49:24.86Z" },
1154  ]
1155  
1156  [[package]]
1157  name = "zipp"
1158  version = "3.20.2"
1159  source = { registry = "https://pypi.org/simple" }
1160  sdist = { url = "https://files.pythonhosted.org/packages/54/bf/5c0000c44ebc80123ecbdddba1f5dcd94a5ada602a9c225d84b5aaa55e86/zipp-3.20.2.tar.gz", hash = "sha256:bc9eb26f4506fda01b81bcde0ca78103b6e62f991b381fec825435c836edbc29", size = 24199, upload-time = "2024-09-13T13:44:16.101Z" }
1161  wheels = [
1162      { url = "https://files.pythonhosted.org/packages/62/8b/5ba542fa83c90e09eac972fc9baca7a88e7e7ca4b221a89251954019308b/zipp-3.20.2-py3-none-any.whl", hash = "sha256:a817ac80d6cf4b23bf7f2828b7cabf326f15a001bea8b1f9b49631780ba28350", size = 9200, upload-time = "2024-09-13T13:44:14.38Z" },
1163  ]
```
./files_to_prompt/__init__.py
```python

```
./files_to_prompt/__main__.py
```python
1  from .cli import cli
2  
3  if __name__ == "__main__":
4      cli()
```
./files_to_prompt/cli.py
````python
  1  import os
  2  import shutil
  3  import subprocess
  4  import sys
  5  from fnmatch import fnmatch
  6  
  7  import click
  8  
  9  global_index = 1
 10  
 11  EXT_TO_LANG = {
 12      "py": "python",
 13      "c": "c",
 14      "cpp": "cpp",
 15      "java": "java",
 16      "js": "javascript",
 17      "ts": "typescript",
 18      "html": "html",
 19      "css": "css",
 20      "xml": "xml",
 21      "json": "json",
 22      "yaml": "yaml",
 23      "yml": "yaml",
 24      "sh": "bash",
 25      "rb": "ruby",
 26  }
 27  
 28  
 29  def should_ignore(path, gitignore_rules):
 30      for rule in gitignore_rules:
 31          if fnmatch(os.path.basename(path), rule):
 32              return True
 33          if os.path.isdir(path) and fnmatch(os.path.basename(path) + "/", rule):
 34              return True
 35      return False
 36  
 37  
 38  def read_gitignore(path):
 39      gitignore_path = os.path.join(path, ".gitignore")
 40      if os.path.isfile(gitignore_path):
 41          with open(gitignore_path, "r") as f:
 42              return [
 43                  line.strip() for line in f if line.strip() and not line.startswith("#")
 44              ]
 45      return []
 46  
 47  
 48  def add_line_numbers(content):
 49      lines = content.splitlines()
 50  
 51      padding = len(str(len(lines)))
 52  
 53      numbered_lines = [f"{i + 1:{padding}}  {line}" for i, line in enumerate(lines)]
 54      return "\n".join(numbered_lines)
 55  
 56  
 57  def print_path(writer, path, content, cxml, markdown, line_numbers):
 58      if cxml:
 59          print_as_xml(writer, path, content, line_numbers)
 60      elif markdown:
 61          print_as_markdown(writer, path, content, line_numbers)
 62      else:
 63          print_default(writer, path, content, line_numbers)
 64  
 65  
 66  def print_default(writer, path, content, line_numbers):
 67      writer(path)
 68      writer("---")
 69      if line_numbers:
 70          content = add_line_numbers(content)
 71      writer(content)
 72      writer("")
 73      writer("---")
 74  
 75  
 76  def print_as_xml(writer, path, content, line_numbers):
 77      global global_index
 78      writer(f'<document index="{global_index}">')
 79      writer(f"<source>{path}</source>")
 80      writer("<document_content>")
 81      if line_numbers:
 82          content = add_line_numbers(content)
 83      writer(content)
 84      writer("</document_content>")
 85      writer("</document>")
 86      global_index += 1
 87  
 88  
 89  def print_as_markdown(writer, path, content, line_numbers):
 90      lang = EXT_TO_LANG.get(path.split(".")[-1], "")
 91      # Figure out how many backticks to use
 92      backticks = "```"
 93      while backticks in content:
 94          backticks += "`"
 95      writer(path)
 96      writer(f"{backticks}{lang}")
 97      if line_numbers:
 98          content = add_line_numbers(content)
 99      writer(content)
100      writer(f"{backticks}")
101  
102  
103  def warn_skipping(path, reason):
104      message = f"Warning: Skipping file {path} ({reason})"
105      click.echo(click.style(message, fg="red"), err=True)
106  
107  
108  def extract_docx(path):
109      if shutil.which("pandoc") is None:
110          warn_skipping(path, "pandoc is not installed")
111          return None
112      try:
113          result = subprocess.run(
114              ["pandoc", "--track-changes=all", path, "-t", "gfm"],
115              capture_output=True,
116              text=True,
117              check=True,
118          )
119          return result.stdout
120      except subprocess.CalledProcessError as e:
121          stderr = e.stderr.strip() if e.stderr else "pandoc failed"
122          warn_skipping(path, stderr)
123          return None
124  
125  
126  def table_to_markdown(table):
127      if not table:
128          return ""
129      max_cols = max(len(row) for row in table if row)
130      normalized = []
131      for row in table:
132          row = row or []
133          normalized.append([cell or "" for cell in row] + [""] * (max_cols - len(row)))
134      header = normalized[0]
135      divider = ["---"] * max_cols
136      body = normalized[1:]
137      lines = [
138          "| " + " | ".join(header) + " |",
139          "| " + " | ".join(divider) + " |",
140      ]
141      for row in body:
142          lines.append("| " + " | ".join(row) + " |")
143      return "\n".join(lines)
144  
145  
146  def extract_pdf(path):
147      try:
148          import pdfplumber
149      except ImportError:
150          warn_skipping(path, "pdfplumber is not installed")
151          return None
152  
153      try:
154          fragments = []
155          with pdfplumber.open(path) as pdf:
156              for page_index, page in enumerate(pdf.pages, start=1):
157                  text = page.extract_text() or ""
158                  if text.strip():
159                      fragments.append(text)
160                  tables = page.extract_tables()
161                  for table_index, table in enumerate(tables, start=1):
162                      md_table = table_to_markdown(table)
163                      if md_table:
164                          fragments.append(
165                              f"Table {table_index} (page {page_index}):\n{md_table}"
166                          )
167          return "\n\n".join(fragments)
168      except Exception as e:
169          warn_skipping(path, str(e))
170          return None
171  
172  
173  def read_file_contents(path):
174      ext = os.path.splitext(path)[1].lower()
175      if ext == ".docx":
176          return extract_docx(path)
177      if ext == ".pdf":
178          return extract_pdf(path)
179      try:
180          with open(path, "r", encoding="utf-8") as f:
181              return f.read()
182      except UnicodeDecodeError:
183          warn_skipping(path, "UnicodeDecodeError")
184      except OSError as e:
185          warn_skipping(path, str(e))
186      return None
187  
188  
189  def process_path(
190      path,
191      extensions,
192      include_hidden,
193      ignore_files_only,
194      ignore_gitignore,
195      gitignore_rules,
196      ignore_patterns,
197      writer,
198      claude_xml,
199      markdown,
200      line_numbers=False,
201  ):
202      if os.path.isfile(path):
203          content = read_file_contents(path)
204          if content is not None:
205              print_path(writer, path, content, claude_xml, markdown, line_numbers)
206      elif os.path.isdir(path):
207          for root, dirs, files in os.walk(path):
208              if not include_hidden:
209                  dirs[:] = [d for d in dirs if not d.startswith(".")]
210                  files = [f for f in files if not f.startswith(".")]
211  
212              if not ignore_gitignore:
213                  gitignore_rules.extend(read_gitignore(root))
214                  dirs[:] = [
215                      d
216                      for d in dirs
217                      if not should_ignore(os.path.join(root, d), gitignore_rules)
218                  ]
219                  files = [
220                      f
221                      for f in files
222                      if not should_ignore(os.path.join(root, f), gitignore_rules)
223                  ]
224  
225              if ignore_patterns:
226                  if not ignore_files_only:
227                      dirs[:] = [
228                          d
229                          for d in dirs
230                          if not any(fnmatch(d, pattern) for pattern in ignore_patterns)
231                      ]
232                  files = [
233                      f
234                      for f in files
235                      if not any(fnmatch(f, pattern) for pattern in ignore_patterns)
236                  ]
237  
238              if extensions:
239                  files = [f for f in files if f.endswith(extensions)]
240  
241              for file in sorted(files):
242                  file_path = os.path.join(root, file)
243                  content = read_file_contents(file_path)
244                  if content is not None:
245                      print_path(
246                          writer,
247                          file_path,
248                          content,
249                          claude_xml,
250                          markdown,
251                          line_numbers,
252                      )
253  
254  
255  def read_paths_from_stdin(use_null_separator):
256      if sys.stdin.isatty():
257          # No ready input from stdin, don't block for input
258          return []
259  
260      stdin_content = sys.stdin.read()
261      if use_null_separator:
262          paths = stdin_content.split("\0")
263      else:
264          paths = stdin_content.split()  # split on whitespace
265      return [p for p in paths if p]
266  
267  
268  @click.command()
269  @click.argument("paths", nargs=-1, type=click.Path(exists=True))
270  @click.option("extensions", "-e", "--extension", multiple=True)
271  @click.option(
272      "--include-hidden",
273      is_flag=True,
274      help="Include files and folders starting with .",
275  )
276  @click.option(
277      "--ignore-files-only",
278      is_flag=True,
279      help="--ignore option only ignores files",
280  )
281  @click.option(
282      "--ignore-gitignore",
283      is_flag=True,
284      help="Ignore .gitignore files and include all files",
285  )
286  @click.option(
287      "ignore_patterns",
288      "--ignore",
289      multiple=True,
290      default=[],
291      help="List of patterns to ignore",
292  )
293  @click.option(
294      "output_file",
295      "-o",
296      "--output",
297      type=click.Path(writable=True),
298      help="Output to a file instead of stdout",
299  )
300  @click.option(
301      "claude_xml",
302      "-c",
303      "--cxml",
304      is_flag=True,
305      help="Output in XML-ish format suitable for Claude's long context window.",
306  )
307  @click.option(
308      "markdown",
309      "-m",
310      "--markdown",
311      is_flag=True,
312      help="Output Markdown with fenced code blocks",
313  )
314  @click.option(
315      "line_numbers",
316      "-n",
317      "--line-numbers",
318      is_flag=True,
319      help="Add line numbers to the output",
320  )
321  @click.option(
322      "--null",
323      "-0",
324      is_flag=True,
325      help="Use NUL character as separator when reading from stdin",
326  )
327  @click.version_option()
328  def cli(
329      paths,
330      extensions,
331      include_hidden,
332      ignore_files_only,
333      ignore_gitignore,
334      ignore_patterns,
335      output_file,
336      claude_xml,
337      markdown,
338      line_numbers,
339      null,
340  ):
341      """
342      Takes one or more paths to files or directories and outputs every file,
343      recursively, each one preceded with its filename like this:
344  
345      \b
346          path/to/file.py
347          ----
348          Contents of file.py goes here
349          ---
350          path/to/file2.py
351          ---
352          ...
353  
354      If the `--cxml` flag is provided, the output will be structured as follows:
355  
356      \b
357          <documents>
358          <document path="path/to/file1.txt">
359          Contents of file1.txt
360          </document>
361          <document path="path/to/file2.txt">
362          Contents of file2.txt
363          </document>
364          ...
365          </documents>
366  
367      If the `--markdown` flag is provided, the output will be structured as follows:
368  
369      \b
370          path/to/file1.py
371          ```python
372          Contents of file1.py
373          ```
374      """
375      # Reset global_index for pytest
376      global global_index
377      global_index = 1
378  
379      # Read paths from stdin if available
380      stdin_paths = read_paths_from_stdin(use_null_separator=null)
381  
382      # Combine paths from arguments and stdin
383      paths = [*paths, *stdin_paths]
384  
385      gitignore_rules = []
386      writer = click.echo
387      fp = None
388      if output_file:
389          fp = open(output_file, "w", encoding="utf-8")
390          writer = lambda s: print(s, file=fp)
391      for path in paths:
392          if not os.path.exists(path):
393              raise click.BadArgumentUsage(f"Path does not exist: {path}")
394          if not ignore_gitignore:
395              gitignore_rules.extend(read_gitignore(os.path.dirname(path)))
396          if claude_xml and path == paths[0]:
397              writer("<documents>")
398          process_path(
399              path,
400              extensions,
401              include_hidden,
402              ignore_files_only,
403              ignore_gitignore,
404              gitignore_rules,
405              ignore_patterns,
406              writer,
407              claude_xml,
408              markdown,
409              line_numbers,
410          )
411      if claude_xml:
412          writer("</documents>")
413      if fp:
414          fp.close()
````
./tests/test_files_to_prompt.py
``````python
  1  import os
  2  import pytest
  3  import re
  4  import shutil
  5  
  6  from click.testing import CliRunner
  7  
  8  from files_to_prompt.cli import cli
  9  
 10  
 11  def filenames_from_cxml(cxml_string):
 12      "Return set of filenames from <source>...</source> tags"
 13      return set(re.findall(r"<source>(.*?)</source>", cxml_string))
 14  
 15  
 16  def test_basic_functionality(tmpdir):
 17      runner = CliRunner()
 18      with tmpdir.as_cwd():
 19          os.makedirs("test_dir")
 20          with open("test_dir/file1.txt", "w") as f:
 21              f.write("Contents of file1")
 22          with open("test_dir/file2.txt", "w") as f:
 23              f.write("Contents of file2")
 24  
 25          result = runner.invoke(cli, ["test_dir"])
 26          assert result.exit_code == 0
 27          assert "test_dir/file1.txt" in result.output
 28          assert "Contents of file1" in result.output
 29          assert "test_dir/file2.txt" in result.output
 30          assert "Contents of file2" in result.output
 31  
 32  
 33  def test_include_hidden(tmpdir):
 34      runner = CliRunner()
 35      with tmpdir.as_cwd():
 36          os.makedirs("test_dir")
 37          with open("test_dir/.hidden.txt", "w") as f:
 38              f.write("Contents of hidden file")
 39  
 40          result = runner.invoke(cli, ["test_dir"])
 41          assert result.exit_code == 0
 42          assert "test_dir/.hidden.txt" not in result.output
 43  
 44          result = runner.invoke(cli, ["test_dir", "--include-hidden"])
 45          assert result.exit_code == 0
 46          assert "test_dir/.hidden.txt" in result.output
 47          assert "Contents of hidden file" in result.output
 48  
 49  
 50  def test_ignore_gitignore(tmpdir):
 51      runner = CliRunner()
 52      with tmpdir.as_cwd():
 53          os.makedirs("test_dir")
 54          os.makedirs("test_dir/nested_include")
 55          os.makedirs("test_dir/nested_ignore")
 56          with open("test_dir/.gitignore", "w") as f:
 57              f.write("ignored.txt")
 58          with open("test_dir/ignored.txt", "w") as f:
 59              f.write("This file should be ignored")
 60          with open("test_dir/included.txt", "w") as f:
 61              f.write("This file should be included")
 62          with open("test_dir/nested_include/included2.txt", "w") as f:
 63              f.write("This nested file should be included")
 64          with open("test_dir/nested_ignore/.gitignore", "w") as f:
 65              f.write("nested_ignore.txt")
 66          with open("test_dir/nested_ignore/nested_ignore.txt", "w") as f:
 67              f.write("This nested file should not be included")
 68          with open("test_dir/nested_ignore/actually_include.txt", "w") as f:
 69              f.write("This nested file should actually be included")
 70  
 71          result = runner.invoke(cli, ["test_dir", "-c"])
 72          assert result.exit_code == 0
 73          filenames = filenames_from_cxml(result.output)
 74  
 75          assert filenames == {
 76              "test_dir/included.txt",
 77              "test_dir/nested_include/included2.txt",
 78              "test_dir/nested_ignore/actually_include.txt",
 79          }
 80  
 81          result2 = runner.invoke(cli, ["test_dir", "-c", "--ignore-gitignore"])
 82          assert result2.exit_code == 0
 83          filenames2 = filenames_from_cxml(result2.output)
 84  
 85          assert filenames2 == {
 86              "test_dir/included.txt",
 87              "test_dir/ignored.txt",
 88              "test_dir/nested_include/included2.txt",
 89              "test_dir/nested_ignore/nested_ignore.txt",
 90              "test_dir/nested_ignore/actually_include.txt",
 91          }
 92  
 93  
 94  def test_multiple_paths(tmpdir):
 95      runner = CliRunner()
 96      with tmpdir.as_cwd():
 97          os.makedirs("test_dir1")
 98          with open("test_dir1/file1.txt", "w") as f:
 99              f.write("Contents of file1")
100          os.makedirs("test_dir2")
101          with open("test_dir2/file2.txt", "w") as f:
102              f.write("Contents of file2")
103          with open("single_file.txt", "w") as f:
104              f.write("Contents of single file")
105  
106          result = runner.invoke(cli, ["test_dir1", "test_dir2", "single_file.txt"])
107          assert result.exit_code == 0
108          assert "test_dir1/file1.txt" in result.output
109          assert "Contents of file1" in result.output
110          assert "test_dir2/file2.txt" in result.output
111          assert "Contents of file2" in result.output
112          assert "single_file.txt" in result.output
113          assert "Contents of single file" in result.output
114  
115  
116  def test_ignore_patterns(tmpdir):
117      runner = CliRunner()
118      with tmpdir.as_cwd():
119          os.makedirs("test_dir", exist_ok=True)
120          with open("test_dir/file_to_ignore.txt", "w") as f:
121              f.write("This file should be ignored due to ignore patterns")
122          with open("test_dir/file_to_include.txt", "w") as f:
123              f.write("This file should be included")
124  
125          result = runner.invoke(cli, ["test_dir", "--ignore", "*.txt"])
126          assert result.exit_code == 0
127          assert "test_dir/file_to_ignore.txt" not in result.output
128          assert "This file should be ignored due to ignore patterns" not in result.output
129          assert "test_dir/file_to_include.txt" not in result.output
130  
131          os.makedirs("test_dir/test_subdir", exist_ok=True)
132          with open("test_dir/test_subdir/any_file.txt", "w") as f:
133              f.write("This entire subdirectory should be ignored due to ignore patterns")
134          result = runner.invoke(cli, ["test_dir", "--ignore", "*subdir*"])
135          assert result.exit_code == 0
136          assert "test_dir/test_subdir/any_file.txt" not in result.output
137          assert (
138              "This entire subdirectory should be ignored due to ignore patterns"
139              not in result.output
140          )
141          assert "test_dir/file_to_include.txt" in result.output
142          assert "This file should be included" in result.output
143          assert "This file should be included" in result.output
144  
145          result = runner.invoke(
146              cli, ["test_dir", "--ignore", "*subdir*", "--ignore-files-only"]
147          )
148          assert result.exit_code == 0
149          assert "test_dir/test_subdir/any_file.txt" in result.output
150  
151          result = runner.invoke(cli, ["test_dir", "--ignore", ""])
152  
153  
154  def test_specific_extensions(tmpdir):
155      runner = CliRunner()
156      with tmpdir.as_cwd():
157          # Write one.txt one.py two/two.txt two/two.py three.md
158          os.makedirs("test_dir/two")
159          with open("test_dir/one.txt", "w") as f:
160              f.write("This is one.txt")
161          with open("test_dir/one.py", "w") as f:
162              f.write("This is one.py")
163          with open("test_dir/two/two.txt", "w") as f:
164              f.write("This is two/two.txt")
165          with open("test_dir/two/two.py", "w") as f:
166              f.write("This is two/two.py")
167          with open("test_dir/three.md", "w") as f:
168              f.write("This is three.md")
169  
170          # Try with -e py -e md
171          result = runner.invoke(cli, ["test_dir", "-e", "py", "-e", "md"])
172          assert result.exit_code == 0
173          assert ".txt" not in result.output
174          assert "test_dir/one.py" in result.output
175          assert "test_dir/two/two.py" in result.output
176          assert "test_dir/three.md" in result.output
177  
178  
179  def test_mixed_paths_with_options(tmpdir):
180      runner = CliRunner()
181      with tmpdir.as_cwd():
182          os.makedirs("test_dir")
183          with open("test_dir/.gitignore", "w") as f:
184              f.write("ignored_in_gitignore.txt\n.hidden_ignored_in_gitignore.txt")
185          with open("test_dir/ignored_in_gitignore.txt", "w") as f:
186              f.write("This file should be ignored by .gitignore")
187          with open("test_dir/.hidden_ignored_in_gitignore.txt", "w") as f:
188              f.write("This hidden file should be ignored by .gitignore")
189          with open("test_dir/included.txt", "w") as f:
190              f.write("This file should be included")
191          with open("test_dir/.hidden_included.txt", "w") as f:
192              f.write("This hidden file should be included")
193          with open("single_file.txt", "w") as f:
194              f.write("Contents of single file")
195  
196          result = runner.invoke(cli, ["test_dir", "single_file.txt"])
197          assert result.exit_code == 0
198          assert "test_dir/ignored_in_gitignore.txt" not in result.output
199          assert "test_dir/.hidden_ignored_in_gitignore.txt" not in result.output
200          assert "test_dir/included.txt" in result.output
201          assert "test_dir/.hidden_included.txt" not in result.output
202          assert "single_file.txt" in result.output
203          assert "Contents of single file" in result.output
204  
205          result = runner.invoke(cli, ["test_dir", "single_file.txt", "--include-hidden"])
206          assert result.exit_code == 0
207          assert "test_dir/ignored_in_gitignore.txt" not in result.output
208          assert "test_dir/.hidden_ignored_in_gitignore.txt" not in result.output
209          assert "test_dir/included.txt" in result.output
210          assert "test_dir/.hidden_included.txt" in result.output
211          assert "single_file.txt" in result.output
212          assert "Contents of single file" in result.output
213  
214          result = runner.invoke(
215              cli, ["test_dir", "single_file.txt", "--ignore-gitignore"]
216          )
217          assert result.exit_code == 0
218          assert "test_dir/ignored_in_gitignore.txt" in result.output
219          assert "test_dir/.hidden_ignored_in_gitignore.txt" not in result.output
220          assert "test_dir/included.txt" in result.output
221          assert "test_dir/.hidden_included.txt" not in result.output
222          assert "single_file.txt" in result.output
223          assert "Contents of single file" in result.output
224  
225          result = runner.invoke(
226              cli,
227              ["test_dir", "single_file.txt", "--ignore-gitignore", "--include-hidden"],
228          )
229          assert result.exit_code == 0
230          assert "test_dir/ignored_in_gitignore.txt" in result.output
231          assert "test_dir/.hidden_ignored_in_gitignore.txt" in result.output
232          assert "test_dir/included.txt" in result.output
233          assert "test_dir/.hidden_included.txt" in result.output
234          assert "single_file.txt" in result.output
235          assert "Contents of single file" in result.output
236  
237  
238  def test_binary_file_warning(tmpdir):
239      runner = CliRunner()
240      with tmpdir.as_cwd():
241          os.makedirs("test_dir")
242          with open("test_dir/binary_file.bin", "wb") as f:
243              f.write(b"\xff")
244          with open("test_dir/text_file.txt", "w") as f:
245              f.write("This is a text file")
246  
247          result = runner.invoke(cli, ["test_dir"])
248          assert result.exit_code == 0
249  
250          stdout = result.stdout
251          stderr = result.stderr
252  
253          assert "test_dir/text_file.txt" in stdout
254          assert "This is a text file" in stdout
255          assert "\ntest_dir/binary_file.bin" not in stdout
256          assert "Skipping file test_dir/binary_file.bin" in stderr
257          assert "UnicodeDecodeError" in stderr
258  
259  
260  @pytest.mark.parametrize(
261      "args", (["test_dir"], ["test_dir/file1.txt", "test_dir/file2.txt"])
262  )
263  def test_xml_format_dir(tmpdir, args):
264      runner = CliRunner()
265      with tmpdir.as_cwd():
266          os.makedirs("test_dir")
267          with open("test_dir/file1.txt", "w") as f:
268              f.write("Contents of file1.txt")
269          with open("test_dir/file2.txt", "w") as f:
270              f.write("Contents of file2.txt")
271          result = runner.invoke(cli, args + ["--cxml"])
272          assert result.exit_code == 0
273          actual = result.output
274          expected = """
275  <documents>
276  <document index="1">
277  <source>test_dir/file1.txt</source>
278  <document_content>
279  Contents of file1.txt
280  </document_content>
281  </document>
282  <document index="2">
283  <source>test_dir/file2.txt</source>
284  <document_content>
285  Contents of file2.txt
286  </document_content>
287  </document>
288  </documents>
289  """
290          assert expected.strip() == actual.strip()
291  
292  
293  @pytest.mark.parametrize("arg", ("-o", "--output"))
294  def test_output_option(tmpdir, arg):
295      runner = CliRunner()
296      with tmpdir.as_cwd():
297          os.makedirs("test_dir")
298          with open("test_dir/file1.txt", "w") as f:
299              f.write("Contents of file1.txt")
300          with open("test_dir/file2.txt", "w") as f:
301              f.write("Contents of file2.txt")
302          output_file = "output.txt"
303          result = runner.invoke(
304              cli, ["test_dir", arg, output_file], catch_exceptions=False
305          )
306          assert result.exit_code == 0
307          assert not result.output
308          with open(output_file, "r") as f:
309              actual = f.read()
310          expected = """
311  test_dir/file1.txt
312  ---
313  Contents of file1.txt
314  
315  ---
316  test_dir/file2.txt
317  ---
318  Contents of file2.txt
319  
320  ---
321  """
322          assert expected.strip() == actual.strip()
323  
324  
325  def test_line_numbers(tmpdir):
326      runner = CliRunner()
327      with tmpdir.as_cwd():
328          os.makedirs("test_dir")
329          test_content = "First line\nSecond line\nThird line\nFourth line\n"
330          with open("test_dir/multiline.txt", "w") as f:
331              f.write(test_content)
332  
333          result = runner.invoke(cli, ["test_dir"])
334          assert result.exit_code == 0
335          assert "1  First line" not in result.output
336          assert test_content in result.output
337  
338          result = runner.invoke(cli, ["test_dir", "-n"])
339          assert result.exit_code == 0
340          assert "1  First line" in result.output
341          assert "2  Second line" in result.output
342          assert "3  Third line" in result.output
343          assert "4  Fourth line" in result.output
344  
345          result = runner.invoke(cli, ["test_dir", "--line-numbers"])
346          assert result.exit_code == 0
347          assert "1  First line" in result.output
348          assert "2  Second line" in result.output
349          assert "3  Third line" in result.output
350          assert "4  Fourth line" in result.output
351  
352  
353  @pytest.mark.parametrize(
354      "input,extra_args",
355      (
356          ("test_dir1/file1.txt\ntest_dir2/file2.txt", []),
357          ("test_dir1/file1.txt\ntest_dir2/file2.txt", []),
358          ("test_dir1/file1.txt\0test_dir2/file2.txt", ["--null"]),
359          ("test_dir1/file1.txt\0test_dir2/file2.txt", ["-0"]),
360      ),
361  )
362  def test_reading_paths_from_stdin(tmpdir, input, extra_args):
363      runner = CliRunner()
364      with tmpdir.as_cwd():
365          # Create test files
366          os.makedirs("test_dir1")
367          os.makedirs("test_dir2")
368          with open("test_dir1/file1.txt", "w") as f:
369              f.write("Contents of file1")
370          with open("test_dir2/file2.txt", "w") as f:
371              f.write("Contents of file2")
372  
373          # Test space-separated paths from stdin
374          result = runner.invoke(cli, args=extra_args, input=input)
375          assert result.exit_code == 0
376          assert "test_dir1/file1.txt" in result.output
377          assert "Contents of file1" in result.output
378          assert "test_dir2/file2.txt" in result.output
379          assert "Contents of file2" in result.output
380  
381  
382  def test_paths_from_arguments_and_stdin(tmpdir):
383      runner = CliRunner()
384      with tmpdir.as_cwd():
385          # Create test files
386          os.makedirs("test_dir1")
387          os.makedirs("test_dir2")
388          with open("test_dir1/file1.txt", "w") as f:
389              f.write("Contents of file1")
390          with open("test_dir2/file2.txt", "w") as f:
391              f.write("Contents of file2")
392  
393          # Test paths from arguments and stdin
394          result = runner.invoke(
395              cli,
396              args=["test_dir1"],
397              input="test_dir2/file2.txt",
398          )
399          assert result.exit_code == 0
400          assert "test_dir1/file1.txt" in result.output
401          assert "Contents of file1" in result.output
402          assert "test_dir2/file2.txt" in result.output
403          assert "Contents of file2" in result.output
404  
405  
406  @pytest.mark.parametrize("option", ("-m", "--markdown"))
407  def test_markdown(tmpdir, option):
408      runner = CliRunner()
409      with tmpdir.as_cwd():
410          os.makedirs("test_dir")
411          with open("test_dir/python.py", "w") as f:
412              f.write("This is python")
413          with open("test_dir/python_with_quad_backticks.py", "w") as f:
414              f.write("This is python with ```` in it already")
415          with open("test_dir/code.js", "w") as f:
416              f.write("This is javascript")
417          with open("test_dir/code.unknown", "w") as f:
418              f.write("This is an unknown file type")
419          result = runner.invoke(cli, ["test_dir", option])
420          assert result.exit_code == 0
421          actual = result.output
422          expected = (
423              "test_dir/code.js\n"
424              "```javascript\n"
425              "This is javascript\n"
426              "```\n"
427              "test_dir/code.unknown\n"
428              "```\n"
429              "This is an unknown file type\n"
430              "```\n"
431              "test_dir/python.py\n"
432              "```python\n"
433              "This is python\n"
434              "```\n"
435              "test_dir/python_with_quad_backticks.py\n"
436              "`````python\n"
437              "This is python with ```` in it already\n"
438              "`````\n"
439          )
440          assert expected.strip() == actual.strip()
441  
442  
443  @pytest.mark.skipif(
444      shutil.which("pandoc") is None, reason="pandoc is required for DOCX extraction"
445  )
446  def test_docx_extraction():
447      runner = CliRunner()
448      result = runner.invoke(cli, ["tests/media/Gemini 3 Developer Guide.docx"])
449      assert result.exit_code == 0
450      assert "tests/media/Gemini 3 Developer Guide.docx" in result.output
451      assert "Gemini 3 Developer Guide" in result.output
452      assert "Gemini 3 is our most intelligent model family to date" in result.output
453  
454  
455  def test_pdf_extraction():
456      pytest.importorskip("pdfplumber")
457      runner = CliRunner()
458      result = runner.invoke(cli, ["tests/media/GOOG 10-Q Q2 summary 2025.pdf"])
459      assert result.exit_code == 0
460      assert "tests/media/GOOG 10-Q Q2 summary 2025.pdf" in result.output
461      assert "UNITED STATES" in result.output
462      assert "Alphabet Inc." in result.output
463      assert "Table 1 (page 6)" in result.output
``````
./tests/media/GOOG 10-Q Q2 summary 2025.pdf
```
  1  UNITED STATES
  2  SECURITIES AND EXCHANGE COMMISSION
  3  Washington, D.C. 20549
  4  ________________________________________________________________________________________
  5  FORM 10-Q
  6  _______________________________________________________________________________________
  7  (Mark One)
  8  ☒ QUARTERLY REPORT PURSUANT TO SECTION 13 OR 15(d) OF THE SECURITIES EXCHANGE ACT OF 1934
  9  For the quarterly period ended June 30, 2025
 10  OR
 11  ☐ TRANSITION REPORT PURSUANT TO SECTION 13 OR 15(d) OF THE SECURITIES EXCHANGE ACT OF 1934
 12  For the transition period from _______ to _______
 13  Commission file number: 001-37580
 14  ________________________________________________________________________________________
 15  Alphabet Inc.
 16  (Exact name of registrant as specified in its charter)
 17  ________________________________________________________________________________________
 18  Delaware 61-1767919
 19  (State or other jurisdiction of incorporation or organization) (I.R.S. Employer Identification Number)
 20  1600 Amphitheatre Parkway
 21  Mountain View, CA 94043
 22  (Address of principal executive offices, including zip code)
 23  (650) 253-0000
 24  (Registrant's telephone number, including area code)
 25  Securities registered pursuant to Section 12(b) of the Act:
 26  Title of each class Trading Symbol(s) Name of each exchange on which registered
 27  Class A Common Stock, $0.001 par value GOOGL Nasdaq Stock Market LLC
 28  (Nasdaq Global Select Market)
 29  Class C Capital Stock, $0.001 par value GOOG Nasdaq Stock Market LLC
 30  (Nasdaq Global Select Market)
 31  2.500% Senior Notes due 2029 — Nasdaq Stock Market LLC
 32  3.000% Senior Notes due 2033 — Nasdaq Stock Market LLC
 33  3.375% Senior Notes due 2037 — Nasdaq Stock Market LLC
 34  3.875% Senior Notes due 2045 — Nasdaq Stock Market LLC
 35  4.000% Senior Notes due 2054 — Nasdaq Stock Market LLC
 36  ________________________________________________________________________________________
 37  Indicate by check mark whether the registrant: (1) has filed all reports required to be filed by Section 13 or 15(d) of the Securities
 38  Exchange Act of 1934 during the preceding 12 months (or for such shorter period that the registrant was required to file such
 39  reports), and (2) has been subject to such filing requirements for the past 90 days. Yes ☒ No ☐
 40  Indicate by check mark whether the registrant has submitted electronically every Interactive Data File required to be submitted
 41  pursuant to Rule 405 of Regulation S-T (§232.405 of this chapter) during the preceding 12 months (or for such shorter period
 42  that the registrant was required to submit such files). Yes ☒ No ☐
 43  Indicate by check mark whether the registrant is a large accelerated filer, an accelerated filer, a non-accelerated filer, a smaller
 44  reporting company, or an emerging growth company. See the definitions of “large accelerated filer,” “accelerated filer,” “smaller
 45  reporting company,” and "emerging growth company" in Rule 12b-2 of the Exchange Act.
 46  Large accelerated filer ☒ Accelerated filer ☐
 47  Non-accelerated filer ☐ Smaller reporting company ☐
 48  Emerging growth company ☐
 49  If an emerging growth company, indicate by check mark if the registrant has elected not to use the extended transition period for
 50  complying with any new or revised financial accounting standards provided pursuant to Section 13(a) of the Exchange Act. ☐
 51  
 52  Indicate by check mark whether the registrant is a shell company (as defined in Rule 12b-2 of the Exchange Act). Yes ☐ No
 53  ☒
 54  As of July 16, 2025, there were 5,817 million shares of Alphabet’s Class A stock outstanding, 847 million shares of Alphabet's
 55  Class B stock outstanding, and 5,430 million shares of Alphabet's Class C stock outstanding.
 56  2
 57  
 58  Table 1 (page 2):
 59  | he registrant is a shell company (as defined in Rule 12b-2 of the Exchange Act). Yes | ☐ | No |
 60  | --- | --- | --- |
 61  
 62  Table 2 (page 2):
 63  | As of | July 16, 2025 | , there were |
 64  | --- | --- | --- |
 65  
 66  Table of Contents Alphabet Inc.
 67  Alphabet Inc.
 68  Form 10-Q
 69  For the Quarterly Period Ended June 30, 2025
 70  TABLE OF CONTENTS
 71  Page No.
 72  Note About Forward-Looking Statements 4
 73  PART I. FINANCIAL INFORMATION
 74  Item 1 Financial Statements 6
 75  Consolidated Balance Sheets - December 31, 2024 and June 30, 2025 6
 76  Consolidated Statements of Income - Three and Six Months Ended June 30, 2024 and 2025 7
 77  Consolidated Statements of Comprehensive Income - Three and Six Months Ended June 30, 8
 78  2024 and 2025
 79  Consolidated Statements of Stockholders' Equity - Three and Six Months Ended June 30, 9
 80  2024 and 2025
 81  Consolidated Statements of Cash Flows - Six Months Ended June 30, 2024 and 2025 11
 82  Notes to Consolidated Financial Statements 12
 83  Item 2 Management’s Discussion and Analysis of Financial Condition and Results of Operations 37
 84  Item 3 Quantitative and Qualitative Disclosures About Market Risk 51
 85  Item 4 Controls and Procedures 52
 86  PART II. OTHER INFORMATION
 87  Item 1 Legal Proceedings 53
 88  Item 1A Risk Factors 53
 89  Item 2 Unregistered Sales of Equity Securities and Use of Proceeds 53
 90  Item 5 Other Information 53
 91  Item 6 Exhibits 55
 92  Signatures 57
 93  3
 94  
 95  Table of Contents Alphabet Inc.
 96  Note About Forward-Looking Statements
 97  This Quarterly Report on Form 10-Q contains forward-looking statements within the meaning of the Private
 98  Securities Litigation Reform Act of 1995. These include, among other things, statements regarding:
 99  • the growth of our business and revenues and our expectations about the factors that influence our success
100  and trends in our business;
101  • fluctuations in our revenues and margins and various factors contributing to such fluctuations;
102  • our expectation that the continuing shift to an online world as the digital economy evolves will continue to
103  benefit our business;
104  • our expectation that the revenues that we derive beyond advertising will continue to increase and may affect
105  our margins;
106  • our expectation that our traffic acquisition costs (TAC) and the associated TAC rate will fluctuate, which
107  could affect our overall margins;
108  • our expectation that our monetization trends will fluctuate, which could affect our revenues and margins;
109  • fluctuations in paid clicks and cost-per-click as well as impressions and cost-per-impression, and various
110  factors contributing to such fluctuations;
111  • our expectation that we will continue to periodically review, refine, and update our methodologies for
112  monitoring, gathering, and counting the number of paid clicks and impressions, and for identifying the
113  revenues generated by the corresponding click and impression activity;
114  • our expectation that our results will be affected by our performance in international markets as users in
115  developing economies increasingly come online;
116  • our expectation that our foreign exchange risk management program will not fully offset our net exposure to
117  fluctuations in foreign currency exchange rates;
118  • the expected variability of gains and losses related to hedging activities under our foreign exchange risk
119  management program;
120  • the amount and timing of revenue recognition from customer contracts with commitments for performance
121  obligations, including our estimate of the remaining amount of commitments and when we expect to
122  recognize revenue;
123  • our expectation that our capital expenditures will increase, including our expected spend and the expected
124  increase in our technical infrastructure investment to support the growth of our business and our long-term
125  initiatives, in particular in support of artificial intelligence (AI) products and services;
126  • our plans to continue to invest in new businesses, products, services and technologies, and systems, as
127  well as to continue to invest in acquisitions and strategic investments;
128  • our pace of hiring and our plans to provide competitive compensation programs;
129  • our expectation that our cost of revenues, research and development (R&D) expenses, sales and marketing
130  expenses, and general and administrative expenses may increase in amount and/or may increase as a
131  percentage of revenues and may be affected by a number of factors;
132  • estimates of our future employee compensation expenses;
133  • our expectation that our other income (expense), net (OI&E), will fluctuate in the future, as it is largely
134  driven by market dynamics;
135  • fluctuations in our effective tax rate;
136  • seasonal fluctuations in internet usage, advertising expenditures, and underlying business trends such as
137  traditional retail seasonality, which are likely to cause fluctuations in our quarterly results;
138  • the sufficiency of our sources of funding;
139  • our potential exposure in connection with new and pending investigations, proceedings, and other
140  contingencies, including the possibility that certain legal proceedings to which we are a party could harm
141  our business, financial condition, and operating results;
142  4
143  
144  Table 1 (page 4):
145  | Note About Forw | ard-Looking Statements |
146  | --- | --- |
147  
148  Table 2 (page 4):
149  | the growth of our business and revenues and our expectations about the factors that influence our success |
150  | --- |
151  | and trends in our business; |
152  
153  Table 3 (page 4):
154  | our expectation that the continuing shift to an online world as the digital economy evolves will continue to |
155  | --- |
156  | benefit our business; |
157  
158  Table 4 (page 4):
159  | our expectation that the revenues that we derive beyond advertising will continue to increase and may affect |
160  | --- |
161  | our margins; |
162  
163  Table 5 (page 4):
164  | our expectation that our traffic acquisition costs (TAC) and the associated TAC rate will fluctuate, which |
165  | --- |
166  | could affect our overall margins; |
167  
168  Table 6 (page 4):
169  | fluctuations in paid clicks and cost-per-click as well as impressions and cost-per-impression, and various |
170  | --- |
171  | factors contributing to such fluctuations; |
172  
173  Table 7 (page 4):
174  | our expectation that we will continue to periodically review, refine, and update our methodologies for |
175  | --- |
176  | monitoring, gathering, and counting the number of paid clicks and impressions, and for identifying the |
177  | revenues generated by the corresponding click and impression activity; |
178  
179  Table 8 (page 4):
180  | our expectation that our results will be affected by our performance in international markets as users in |
181  | --- |
182  | developing economies increasingly come online; |
183  
184  Table 9 (page 4):
185  | our expectation that our foreign exchange risk management program will not fully offset our net exposure to |
186  | --- |
187  | fluctuations in foreign currency exchange rates; |
188  
189  Table 10 (page 4):
190  | the expected variability of gains and losses related to hedging activities under our foreign exchange risk |
191  | --- |
192  | management program; |
193  
194  Table 11 (page 4):
195  | the amount and timing of revenue recognition from customer contracts with commitments for performance |
196  | --- |
197  | obligations, including our estimate of the remaining amount of commitments and when we expect to |
198  | recognize revenue; |
199  
200  Table 12 (page 4):
201  | our expectation that our capital expenditures will increase, including our expected spend and the expected |
202  | --- |
203  | increase in our technical infrastructure investment to support the growth of our business and our long-term |
204  | initiatives, in particular in support of artificial intelligence (AI) products and services; |
205  
206  Table 13 (page 4):
207  | our plans to continue to invest in new businesses, products, services and technologies, and systems, as |
208  | --- |
209  | well as to continue to invest in acquisitions and strategic investments; |
210  
211  Table 14 (page 4):
212  | our expectation that our cost of revenues, research and development (R&D) expenses, sales and marketing |
213  | --- |
214  | expenses, and general and administrative expenses may increase in amount and/or may increase as a |
215  | percentage of revenues and may be affected by a number of factors; |
216  
217  Table 15 (page 4):
218  | our expectation that our other income (expense), net (OI&E), will fluctuate in the future, as it is largely |
219  | --- |
220  | driven by market dynamics; |
221  
222  Table 16 (page 4):
223  | fluctuations in | our effective tax rat | e; |
224  | --- | --- | --- |
225  
226  Table 17 (page 4):
227  | seasonal fluctuations in internet usage, advertising expenditures, and underlying business trends such as |
228  | --- |
229  | traditional retail seasonality, which are likely to cause fluctuations in our quarterly results; |
230  
231  Table 18 (page 4):
232  | our potential exposure in connection with new and pending investigations, proceedings, and other |
233  | --- |
234  | contingencies, including the possibility that certain legal proceedings to which we are a party could harm |
235  | our business, financial condition, and operating results; |
236  
237  Table of Contents Alphabet Inc.
238  • our expectation that we will continue to face heightened regulatory scrutiny, and changes in regulatory
239  conditions, laws, and public policies, which could affect our business practices and financial results;
240  • the expected timing, amount, and effect of Alphabet Inc.'s share repurchases and dividends;
241  • our long-term sustainability goals;
242  • our expectations regarding the timing and successful closing and integration of the Wiz, Inc. ("Wiz")
243  acquisition, including the realization of anticipated benefits; and
244  • ongoing developments surrounding international trade and the related impact on the macroeconomic
245  environment and our business;
246  as well as other statements regarding our future operations, financial condition and prospects, and business
247  strategies. Forward-looking statements may appear throughout this report and other documents we file with the
248  Securities and Exchange Commission (SEC), including without limitation, the following sections: Part I, Item 2,
249  "Management's Discussion and Analysis of Financial Condition and Results of Operations" in this Quarterly Report
250  on Form 10-Q and Part I, Item 1A, “Risk Factors” in our Annual Report on Form 10-K for the fiscal year ended
251  December 31, 2024. Forward-looking statements generally can be identified by words such as "anticipates,"
252  "believes," "could," "estimates," "expects," "intends," "may," "plans," "predicts," "projects," "will be," "will continue,"
253  "will likely result," and similar expressions. These forward-looking statements are based on current expectations and
254  assumptions that are subject to risks and uncertainties, which could cause our actual results to differ materially from
255  those reflected in the forward-looking statements. Factors that could cause or contribute to such differences include,
256  but are not limited to, those discussed in this Quarterly Report on Form 10-Q; the risks discussed in Part I, Item 1A,
257  "Risk Factors" and the trends discussed in Part II, Item 7, "Management's Discussion and Analysis of Financial
258  Condition and Results of Operations" in our Annual Report on Form 10-K for the fiscal year ended December 31,
259  2024; and those discussed in other documents we file with the SEC. We undertake no obligation to revise or
260  publicly release the results of any revision to these forward-looking statements, except as required by law. Given
261  these risks and uncertainties, readers are cautioned not to place undue reliance on such forward-looking
262  statements.
263  As used herein, "Alphabet," "the company," "we," "us," "our," and similar terms include Alphabet Inc. and its
264  subsidiaries, unless the context indicates otherwise.
265  "Alphabet," "Google," and other trademarks of ours appearing in this report are our property. We do not intend
266  our use or display of other companies' trade names or trademarks to imply an endorsement or sponsorship of us by
267  such companies, or any relationship with any of these companies.
268  5
269  
270  Table 1 (page 5):
271  | our expectation that we will continue to face heightened regulatory scrutiny, and changes in regulatory |
272  | --- |
273  | conditions, laws, and public policies, which could affect our business practices and financial results; |
274  
275  Table 2 (page 5):
276  | our expectations regarding the timing and successful closing and integration of the Wiz, Inc. ("Wiz") |
277  | --- |
278  | acquisition, including the realization of anticipated benefits; and |
279  
280  Table 3 (page 5):
281  | ongoing developments surrounding international trade and the related impact on the macroeconomic |
282  | --- |
283  | environment and our business; |
284  
285  Table of Contents Alphabet Inc.
286  PART I. FINANCIAL INFORMATION
287  ITEM 1. FINANCIAL STATEMENTS
288  Alphabet Inc.
289  CONSOLIDATED BALANCE SHEETS
290  (in millions, except par value per share amounts)
291  As of As of
292  December 31, 2024 June 30, 2025
293  (unaudited)
294  Assets
295  Current assets:
296  Cash and cash equivalents $ 23,466 $ 21,036
297  Marketable securities 72,191 74,112
298  Total cash, cash equivalents, and marketable securities 95,657 95,148
299  Accounts receivable, net 52,340 55,048
300  Other current assets 15,714 16,020
301  Total current assets 163,711 166,216
302  Non-marketable securities 37,982 52,574
303  Deferred income taxes 17,180 19,289
304  Property and equipment, net 171,036 203,231
305  Operating lease assets 13,588 14,255
306  Goodwill 31,885 32,335
307  Other non-current assets 14,874 14,153
308  Total assets $ 450,256 $ 502,053
309  Liabilities and Stockholders’ Equity
310  Current liabilities:
311  Accounts payable $ 7,987 $ 8,347
312  Accrued compensation and benefits 15,069 12,168
313  Accrued expenses and other current liabilities 51,228 52,039
314  Accrued revenue share 9,802 9,787
315  Deferred revenue 5,036 4,969
316  Total current liabilities 89,122 87,310
317  Long-term debt 10,883 23,607
318  Income taxes payable, non-current 8,782 10,027
319  Operating lease liabilities 11,691 11,952
320  Other long-term liabilities 4,694 6,241
321  Total liabilities 125,172 139,137
322  Commitments and Contingencies (Note 10)
323  Stockholders’ equity:
324  Preferred stock, $0.001 par value per share, 100 shares authorized; no
325  0 0
326  shares issued and outstanding
327  Class A, Class B, and Class C stock and additional paid-in capital,
328  $0.001 par value per share: 300,000 shares authorized (Class A 180,000,
329  Class B 60,000, Class C 60,000); 12,211 (Class A 5,835, Class B 861, 84,800 89,283
330  Class C 5,515) and 12,104 (Class A 5,816, Class B 849, Class C 5,439)
331  shares issued and outstanding
332  Accumulated other comprehensive income (loss) (4,800) (2,127)
333  Retained earnings 245,084 275,760
334  Total stockholders’ equity 325,084 362,916
335  Total liabilities and stockholders’ equity $ 450,256 $ 502,053
336  See accompanying notes.
337  6
338  
339  Table 1 (page 6):
340  | Assets |  |  |
341  | --- | --- | --- |
342  | Current assets: |  |  |
343  | Cash and cash equivalents $ 23,466 $ 21,036 |  |  |
344  | Marketable securities 72,191 74,112 |  |  |
345  | Total cash, cash equivalents, and marketable securities 95,657 95,148 |  |  |
346  | Accounts receivable, net | 52,340 55,048 |  |
347  | Other current assets 15,714 16,020 |  |  |
348  | Total current assets | 163,711 | 166,216 |
349  | Non-marketable securities 37,982 52,574 |  |  |
350  | Deferred income taxes 17,180 19,289 |  |  |
351  | Property and equipment, net 171,036 203,231 |  |  |
352  | Operating lease assets 13,588 14,255 |  |  |
353  | Goodwill 31,885 32,335 |  |  |
354  | Other non-current assets 14,874 14,153 |  |  |
355  | Total assets | $ 450,256 | $ 502,053 |
356  | Liabilities and Stockholders’ Equity |  |  |
357  | Current liabilities: |  |  |
358  | Accounts payable $ 7,987 $ 8,347 |  |  |
359  | Accrued compensation and benefits 15,069 12,168 |  |  |
360  | Accrued expenses and other current liabilities 51,228 52,039 |  |  |
361  | Accrued revenue share 9,802 9,787 |  |  |
362  | Deferred revenue 5,036 4,969 |  |  |
363  | Total current liabilities 89,122 87,310 |  |  |
364  | Long-term debt 10,883 23,607 |  |  |
365  | Income taxes payable, non-current 8,782 10,027 |  |  |
366  | Operating lease liabilities 11,691 11,952 |  |  |
367  | Other long-term liabilities 4,694 6,241 |  |  |
368  | Total liabilities 125,172 139,137 |  |  |
369  | Commitments and Contingencies (Note 10) |  |  |
370  | Stockholders’ equity: |  |  |
371  | Preferred stock, $0.001 par value per share, 100 shares authorized; no
372  0 0
373  shares issued and outstanding |  |  |
374  | Class A, Class B, and Class C stock and additional paid-in capital,
375  $0.001 par value per share: 300,000 shares authorized (Class A 180,000,
376  Class B 60,000, Class C 60,000); 12,211 (Class A 5,835, Class B 861, 84,800 89,283
377  Class C 5,515) and 12,104 (Class A 5,816, Class B 849, Class C 5,439)
378  shares issued and outstanding |  |  |
379  | Accumulated other comprehensive income (loss) (4,800) (2,127) |  |  |
380  | Retained earnings 245,084 275,760 |  |  |
381  | Total stockholders’ equity 325,084 362,916 |  |  |
382  | Total liabilities and stockholders’ equity $ 450,256 $ 502,053 |  |  |
383  
384  Table 2 (page 6):
385  | and | 12,104 | (Class A | 5,816 | , Class B | 849 | , Class C | 5,439 | ) |
386  | --- | --- | --- | --- | --- | --- | --- | --- | --- |
387  
388  Table of Contents Alphabet Inc.
389  Alphabet Inc.
390  CONSOLIDATED STATEMENTS OF INCOME
391  (in millions, except per share amounts; unaudited)
392  Three Months Ended Six Months Ended
393  June 30, June 30,
394  2024 2025 2024 2025
395  Revenues $ 84,742 $ 96,428 $ 165,281 $ 186,662
396  Costs and expenses:
397  Cost of revenues 35,507 39,039 69,219 75,400
398  Research and development 11,860 13,808 23,763 27,364
399  Sales and marketing 6,792 7,101 13,218 13,273
400  General and administrative 3,158 5,209 6,184 8,748
401  Total costs and expenses 57,317 65,157 112,384 124,785
402  Income from operations 27,425 31,271 52,897 61,877
403  Other income (expense), net 126 2,662 2,969 13,845
404  Income before income taxes 27,551 33,933 55,866 75,722
405  Provision for income taxes 3,932 5,737 8,585 12,986
406  Net income $ 23,619 $ 28,196 $ 47,281 $ 62,736
407  Basic net income per share (Note 12) $ 1.91 $ 2.33 $ 3.82 $ 5.16
408  Diluted net income per share (Note 12) $ 1.89 $ 2.31 $ 3.78 $ 5.12
409  See accompanying notes.
410  7
411  
412  Table 1 (page 7):
413  | 2024 | 2025 | 2024 |
414  | --- | --- | --- |
415  
416  Table 2 (page 7):
417  | Revenues $ 84,742 $ 96,428 $ 165,281 $ 186,662 |
418  | --- |
419  | Costs and expenses: |
420  | Cost of revenues 35,507 39,039 69,219 75,400 |
421  | Research and development 11,860 13,808 23,763 27,364 |
422  | Sales and marketing 6,792 7,101 13,218 13,273 |
423  | General and administrative 3,158 5,209 6,184 8,748 |
424  | Total costs and expenses 57,317 65,157 112,384 124,785 |
425  | Income from operations 27,425 31,271 52,897 61,877 |
426  | Other income (expense), net 126 2,662 2,969 13,845 |
427  | Income before income taxes 27,551 33,933 55,866 75,722 |
428  | Provision for income taxes 3,932 5,737 8,585 12,986 |
429  
430  Table 3 (page 7):
431  |  |  |  |
432  | --- | --- | --- |
433  | Basic net income per share (Note 12) $ 1.91 $ 2.33 $ 3.82 $ 5.16 |  |  |
434  | Diluted net income per share (Note 12) | $ 1.89 | $ 2.31 $ 3.78 $ 5.12 |
435  
436  Table of Contents Alphabet Inc.
437  Alphabet Inc.
438  CONSOLIDATED STATEMENTS OF COMPREHENSIVE INCOME
439  (in millions; unaudited)
440  Three Months Ended Six Months Ended
441  June 30, June 30,
442  2024 2025 2024 2025
443  Net income $ 23,619 $ 28,196 $ 47,281 $ 62,736
444  Other comprehensive income (loss):
445  Change in foreign currency translation adjustment, net of income tax
446  benefit (expense) of $(26), $190, $(44), and $235 (447) 2,610 (950) 3,273
447  Available-for-sale investments:
448  Change in net unrealized gains (losses) (93) 191 (453) 836
449  Less: reclassification adjustment for net (gains) losses
450  included in net income 230 (29) 541 (113)
451  Net change, net of income tax benefit (expense) of $(40), $(46),
452  $(26), and $(205) 137 162 88 723
453  Cash flow hedges:
454  Change in net unrealized gains (losses) 232 (920) 418 (1,233)
455  Less: reclassification adjustment for net (gains) losses
456  included in net income (95) 107 (166) (90)
457  Net change, net of income tax benefit (expense) of $(27), $208,
458  $(50), and $339 137 (813) 252 (1,323)
459  Other comprehensive income (loss) (173) 1,959 (610) 2,673
460  Comprehensive income $ 23,446 $ 30,155 $ 46,671 $ 65,409
461  See accompanying notes.
462  8
463  
464  Table 1 (page 8):
465  | Net income $ 23,619 $ 28,196 $ 47,281 $ 62,736 |
466  | --- |
467  | Other comprehensive income (loss): |
468  | Change in foreign currency translation adjustment, net of income tax
469  benefit (expense) of $(26), $190, $(44), and $235 (447) 2,610 (950) 3,273 |
470  | Available-for-sale investments: |
471  | Change in net unrealized gains (losses) (93) 191 (453) 836 |
472  | Less: reclassification adjustment for net (gains) losses
473  included in net income 230 (29) 541 (113) |
474  | Net change, net of income tax benefit (expense) of $(40), $(46),
475  $(26), and $(205) 137 162 88 723 |
476  | Cash flow hedges: |
477  | Change in net unrealized gains (losses) 232 (920) 418 (1,233) |
478  | Less: reclassification adjustment for net (gains) losses
479  included in net income (95) 107 (166) (90) |
480  | Net change, net of income tax benefit (expense) of $(27), $208,
481  $(50), and $339 137 (813) 252 (1,323) |
482  | Other comprehensive income (loss) (173) 1,959 (610) 2,673 |
483  | Comprehensive income $ 23,446 $ 30,155 $ 46,671 $ 65,409 |
484  
485  Table of Contents Alphabet Inc.
486  Alphabet Inc.
487  CONSOLIDATED STATEMENTS OF STOCKHOLDERS’ EQUITY
488  (in millions; unaudited)
489  Three Months Ended June 30, 2024
490  Accumulated
491  Class A, Class B, Class C Stock
492  and Additional Paid-In Capital
493  Other Total
494  Comprehensive Retained Stockholders’
495  Shares Amount Income (Loss) Earnings Equity
496  Balance as of March 31, 2024 12,381 $ 77,913 $ (4,839) $ 219,770 $ 292,844
497  Stock issued 33 0 0 0 0
498  Stock-based compensation 0 5,908 0 0 5,908
499  Tax withholding related to vesting of
500  restricted stock units and other 0 (3,304) 0 0 (3,304)
501  Repurchases of stock (92) (789) 0 (14,809) (15,598)
502  Dividends and dividend equivalents
503  declared ($0.20 per share) 0 4 0 (2,547) (2,543)
504  Net income 0 0 0 23,619 23,619
505  Other comprehensive income (loss) 0 0 (173) 0 (173)
506  Balance as of June 30, 2024 12,322 $ 79,732 $ (5,012) $ 226,033 $ 300,753
507  Six Months Ended June 30, 2024
508  Accumulated
509  Class A, Class B, Class C Stock
510  and Additional Paid-In Capital
511  Other Total
512  Comprehensive Retained Stockholders’
513  Shares Amount Income (Loss) Earnings Equity
514  Balance as of December 31, 2023 12,460 $ 76,534 $ (4,402) $ 211,247 $ 283,379
515  Stock issued 65 0 0 0 0
516  Stock-based compensation 0 11,201 0 0 11,201
517  Tax withholding related to vesting of
518  restricted stock units and other 0 (6,300) 0 0 (6,300)
519  Repurchases of stock (203) (1,707) 0 (29,948) (31,655)
520  Dividends and dividend equivalents
521  declared ($0.20 per share) 0 4 0 (2,547) (2,543)
522  Net income 0 0 0 47,281 47,281
523  Other comprehensive income (loss) 0 0 (610) 0 (610)
524  Balance as of June 30, 2024 12,322 $ 79,732 $ (5,012) $ 226,033 $ 300,753
525  9
526  
527  Table 1 (page 9):
528  |  | Three Months Ended June 30, 2024 |
529  | --- | --- |
530  
531  Table 2 (page 9):
532  | Balance as of March 31, 2024 12,381 $ 77,913 $ (4,839) $ 219,770 $ 292,844 |  |  |  |  |  |
533  | --- | --- | --- | --- | --- | --- |
534  | Stock issued 33 0 0 0 0 |  |  |  |  |  |
535  | Stock-based compensation 0 5,908 0 0 5,908 |  |  |  |  |  |
536  | Tax withholding related to vesting of
537  restricted stock units and other 0 (3,304) 0 0 (3,304) |  |  |  |  |  |
538  | Repurchases of stock (92) (789) 0 (14,809) (15,598) |  |  |  |  |  |
539  | Dividends and dividend equivalents
540  declared ($0.20 per share) 0 4 0 (2,547) (2,543) |  |  |  |  |  |
541  | Net income 0 0 0 23,619 23,619 |  |  |  |  |  |
542  | Other comprehensive income (loss) 0 0 (173) 0 (173) |  |  |  |  |  |
543  | Balance as of June 30, 2024 | 12,322 | $ 79,732 | $ (5,012) | $ 226,033 | $ 300,753 |
544  
545  Table 3 (page 9):
546  |  | Six Months Ended June 30, 2024 |
547  | --- | --- |
548  
549  Table 4 (page 9):
550  | Balance as of December 31, 2023 12,460 $ 76,534 $ (4,402) $ 211,247 $ 283,379 |  |  |  |  |  |
551  | --- | --- | --- | --- | --- | --- |
552  | Stock issued | 65 | 0 | 0 | 0 | 0 |
553  | Stock-based compensation | 0 11,201 0 0 |  |  |  | 11,201 |
554  | Tax withholding related to vesting of
555  restricted stock units and other | 0 | (6,300) | 0 | 0 | (6,300) |
556  | Repurchases of stock | (203) (1,707) 0 (29,948) |  |  |  | (31,655) |
557  | Dividends and dividend equivalents
558  declared ($0.20 per share) 0 4 0 (2,547) (2,543) |  |  |  |  |  |
559  | Net income 0 0 0 47,281 47,281 |  |  |  |  |  |
560  | Other comprehensive income (loss) 0 0 (610) 0 (610) |  |  |  |  |  |
561  | Balance as of June 30, 2024 12,322 $ 79,732 $ (5,012) $ 226,033 $ 300,753 |  |  |  |  |  |
562  
563  Table of Contents Alphabet Inc.
564  Three Months Ended June 30, 2025
565  Accumulated
566  Class A, Class B, Class C Stock
567  and Additional Paid-In Capital
568  Other Total
569  Comprehensive Retained Stockholders’
570  Shares Amount Income (Loss) Earnings Equity
571  Balance as of March 31, 2025 12,155 $ 86,725 $ (4,086) $ 262,628 $ 345,267
572  Stock issued 30 0 0 0 0
573  Stock-based compensation 0 6,045 0 0 6,045
574  Tax withholding related to vesting of
575  restricted stock units and other 0 (2,709) 0 0 (2,709)
576  Repurchases of stock (81) (811) 0 (12,452) (13,263)
577  Dividends and dividend equivalents
578  declared ($0.21 per share) 0 33 0 (2,612) (2,579)
579  Sale of interest in consolidated
580  entities 0 0 0 0 0
581  Net income 0 0 0 28,196 28,196
582  Other comprehensive income (loss) 0 0 1,959 0 1,959
583  Balance as of June 30, 2025 12,104 $ 89,283 $ (2,127) $ 275,760 $ 362,916
584  Six Months Ended June 30, 2025
585  Accumulated
586  Class A, Class B, Class C Stock
587  and Additional Paid-In Capital
588  Other Total
589  Comprehensive Retained Stockholders’
590  Shares Amount Income (Loss) Earnings Equity
591  Balance as of December 31, 2024 12,211 $ 84,800 $ (4,800) $ 245,084 $ 325,084
592  Stock issued 57 0 0 0 0
593  Stock-based compensation 0 11,598 0 0 11,598
594  Tax withholding related to vesting of
595  restricted stock units and other 0 (5,949) 0 0 (5,949)
596  Repurchases of stock (164) (1,626) 0 (26,938) (28,564)
597  Dividends and dividend equivalents
598  declared ($0.41 per share) 0 60 0 (5,122) (5,062)
599  Sale of interest in consolidated
600  entities 0 400 0 0 400
601  Net income 0 0 0 62,736 62,736
602  Other comprehensive income (loss) 0 0 2,673 0 2,673
603  Balance as of June 30, 2025 12,104 $ 89,283 $ (2,127) $ 275,760 $ 362,916
604  See accompanying notes.
605  10
606  
607  Table 1 (page 10):
608  | Balance as of March 31, 2025 12,155 $ 86,725 $ (4,086) $ 262,628 $ 345,267 |
609  | --- |
610  | Stock issued 30 0 0 0 0 |
611  | Stock-based compensation 0 6,045 0 0 6,045 |
612  | Tax withholding related to vesting of
613  restricted stock units and other 0 (2,709) 0 0 (2,709) |
614  | Repurchases of stock (81) (811) 0 (12,452) (13,263) |
615  | Dividends and dividend equivalents
616  declared ($0.21 per share) 0 33 0 (2,612) (2,579) |
617  | Sale of interest in consolidated
618  entities 0 0 0 0 0 |
619  | Net income 0 0 0 28,196 28,196 |
620  | Other comprehensive income (loss) 0 0 1,959 0 1,959 |
621  
622  Table 2 (page 10):
623  | d ( | $0.21 | per sh |
624  | --- | --- | --- |
625  
626  Table 3 (page 10):
627  | Balance as of December 31, 2024 12,211 $ 84,800 $ (4,800) $ 245,084 $ 325,084 |
628  | --- |
629  | Stock issued 57 0 0 0 0 |
630  | Stock-based compensation 0 11,598 0 0 11,598 |
631  | Tax withholding related to vesting of
632  restricted stock units and other 0 (5,949) 0 0 (5,949) |
633  | Repurchases of stock (164) (1,626) 0 (26,938) (28,564) |
634  | Dividends and dividend equivalents
635  declared ($0.41 per share) 0 60 0 (5,122) (5,062) |
636  | Sale of interest in consolidated
637  entities 0 400 0 0 400 |
638  | Net income 0 0 0 62,736 62,736 |
639  | Other comprehensive income (loss) 0 0 2,673 0 2,673 |
640  | Balance as of June 30, 2025 12,104 $ 89,283 $ (2,127) $ 275,760 $ 362,916 |
641  
642  Table 4 (page 10):
643  | red ( | $0.41 |
644  | --- | --- |
645  
646  Table of Contents Alphabet Inc.
647  Alphabet Inc.
648  CONSOLIDATED STATEMENTS OF CASH FLOWS
649  (in millions; unaudited)
650  Six Months Ended
651  June 30,
652  2024 2025
653  Operating activities
654  Net income $ 47,281 $ 62,736
655  Adjustments:
656  Depreciation of property and equipment 7,121 9,485
657  Stock-based compensation expense 11,129 11,514
658  Deferred income taxes (2,738) (1,596)
659  Loss (gain) on debt and equity securities, net (757) (11,411)
660  Other 1,185 1,041
661  Changes in assets and liabilities, net of effects of acquisitions:
662  Accounts receivable, net 110 (1,201)
663  Income taxes, net (889) (2,434)
664  Other assets (1,532) (2,767)
665  Accounts payable (563) (327)
666  Accrued expenses and other liabilities (5,176) (1,560)
667  Accrued revenue share 97 (219)
668  Deferred revenue 220 636
669  Net cash provided by operating activities 55,488 63,897
670  Investing activities
671  Purchases of property and equipment (25,198) (39,643)
672  Purchases of marketable securities (43,011) (39,870)
673  Maturities and sales of marketable securities 58,577 40,930
674  Purchases of non-marketable securities (2,199) (2,312)
675  Maturities and sales of non-marketable securities 605 873
676  Acquisitions, net of cash acquired, and purchases of intangible assets (87) (353)
677  Other investing activities (32) (363)
678  Net cash used in investing activities (11,345) (40,738)
679  Financing activities
680  Net payments related to stock-based award activities (6,138) (5,731)
681  Repurchases of stock (31,380) (28,706)
682  Dividend payments (2,466) (4,977)
683  Proceeds from issuance of debt, net of costs 4,875 31,378
684  Repayments of debt (5,502) (18,397)
685  Proceeds from sale of interest in consolidated entities, net 8 400
686  Net cash used in financing activities (40,603) (26,033)
687  Effect of exchange rate changes on cash and cash equivalents (363) 444
688  Net increase (decrease) in cash and cash equivalents 3,177 (2,430)
689  Cash and cash equivalents at beginning of period 24,048 23,466
690  Cash and cash equivalents at end of period $ 27,225 $ 21,036
691  See accompanying notes.
692  11
693  
694  Table 1 (page 11):
695  | Operating activities |
696  | --- |
697  | Net income $ 47,281 $ 62,736 |
698  | Adjustments: |
699  | Depreciation of property and equipment 7,121 9,485 |
700  | Stock-based compensation expense 11,129 11,514 |
701  | Deferred income taxes (2,738) (1,596) |
702  | Loss (gain) on debt and equity securities, net (757) (11,411) |
703  | Other 1,185 1,041 |
704  | Changes in assets and liabilities, net of effects of acquisitions: |
705  | Accounts receivable, net 110 (1,201) |
706  | Income taxes, net (889) (2,434) |
707  | Other assets (1,532) (2,767) |
708  | Accounts payable (563) (327) |
709  | Accrued expenses and other liabilities (5,176) (1,560) |
710  | Accrued revenue share 97 (219) |
711  | Deferred revenue 220 636 |
712  | Net cash provided by operating activities 55,488 63,897 |
713  | Investing activities |
714  | Purchases of property and equipment (25,198) (39,643) |
715  | Purchases of marketable securities (43,011) (39,870) |
716  | Maturities and sales of marketable securities 58,577 40,930 |
717  | Purchases of non-marketable securities (2,199) (2,312) |
718  | Maturities and sales of non-marketable securities 605 873 |
719  | Acquisitions, net of cash acquired, and purchases of intangible assets (87) (353) |
720  | Other investing activities (32) (363) |
721  | Net cash used in investing activities (11,345) (40,738) |
722  | Financing activities |
723  | Net payments related to stock-based award activities (6,138) (5,731) |
724  | Repurchases of stock (31,380) (28,706) |
725  | Dividend payments (2,466) (4,977) |
726  | Proceeds from issuance of debt, net of costs 4,875 31,378 |
727  | Repayments of debt (5,502) (18,397) |
728  | Proceeds from sale of interest in consolidated entities, net 8 400 |
729  | Net cash used in financing activities (40,603) (26,033) |
730  | Effect of exchange rate changes on cash and cash equivalents (363) 444 |
731  | Net increase (decrease) in cash and cash equivalents 3,177 (2,430) |
732  | Cash and cash equivalents at beginning of period 24,048 23,466 |
733  | Cash and cash equivalents at end of period $ 27,225 $ 21,036 |
```
./tests/media/Gemini 3 Developer Guide.docx
```
  1  Gemini 3 Developer Guide
  2  
  3  content_copy
  4  
  5  Gemini 3 is our most intelligent model family to date, built on a
  6  foundation of state-of-the-art reasoning. It is designed to bring any
  7  idea to life by mastering agentic workflows, autonomous coding, and
  8  complex multimodal tasks. This guide covers key features of the Gemini 3
  9  model family and how to get the most out of it.
 10  
 11  High/Dynamic Thinking Low Thinking
 12  
 13  Gemini 3 Pro uses dynamic thinking by default to reason through prompts.
 14  For faster, lower-latency responses when complex reasoning isn't
 15  required, you can constrain the model's thinking level to low.
 16  
 17  <u>[Python](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#python)[JavaScript](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#javascript)[REST](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#rest)</u>
 18  
 19  from google import genai
 20  
 21  from google.genai import types
 22  
 23  client = genai.Client()
 24  
 25  response = client.models.generate_content(
 26  
 27  model="gemini-3-pro-preview",
 28  
 29  contents="Find the race condition in this multi-threaded C++ snippet:
 30  \[code here\]",
 31  
 32  )
 33  
 34  print(response.text)
 35  
 36  Explore
 37  
 38  <img src="media/image1.png" style="width:6.5in;height:2.46181in"
 39  alt="Gemini 3 Applets Overview" />
 40  
 41  Explore our [<u>collection of Gemini 3
 42  apps</u>](https://aistudio.google.com/app/apps?source=showcase&showcaseTag=gemini-3) to
 43  see how the model handles advanced reasoning, autonomous coding, and
 44  complex multimodal tasks.
 45  
 46  Meet Gemini 3
 47  
 48  Gemini 3 Pro is the first model in the new
 49  series. gemini-3-pro-preview is best for your complex tasks that require
 50  broad world knowledge and advanced reasoning across modalities.
 51  
 52  <table>
 53  <colgroup>
 54  <col style="width: 24%" />
 55  <col style="width: 22%" />
 56  <col style="width: 16%" />
 57  <col style="width: 36%" />
 58  </colgroup>
 59  <thead>
 60  <tr>
 61  <th>Model ID</th>
 62  <th>Context Window (In / Out)</th>
 63  <th>Knowledge Cutoff</th>
 64  <th>Pricing (Input / Output)*</th>
 65  </tr>
 66  </thead>
 67  <tbody>
 68  <tr>
 69  <td>gemini-3-pro-preview</td>
 70  <td>1M / 64k</td>
 71  <td>Jan 2025</td>
 72  <td>$2 / $12 (&lt;200k tokens)<br />
 73  $4 / $18 (&gt;200k tokens)</td>
 74  </tr>
 75  <tr>
 76  <td>gemini-3-pro-image-preview</td>
 77  <td>65k / 32k</td>
 78  <td>Jan 2025</td>
 79  <td>$2 (Text Input) / $0.134 (Image Output)**</td>
 80  </tr>
 81  </tbody>
 82  </table>
 83  
 84  *\* Pricing is per 1 million tokens unless otherwise noted.* *\*\* Image
 85  pricing varies by resolution. See the [<u>pricing
 86  page</u>](https://ai.google.dev/gemini-api/docs/pricing) for details.*
 87  
 88  For detailed rate limits, batch pricing, and additional information, see
 89  the [<u>models
 90  page</u>](https://ai.google.dev/gemini-api/docs/models/gemini).
 91  
 92  New API features in Gemini 3
 93  
 94  Gemini 3 introduces new parameters designed to give developers more
 95  control over latency, cost, and multimodal fidelity.
 96  
 97  Thinking level
 98  
 99  The thinking_level parameter controls the **maximum** depth of the
100  model's internal reasoning process before it produces a response. Gemini
101  3 treats these levels as relative allowances for thinking rather than
102  strict token guarantees. If thinking_level is not specified, Gemini 3
103  Pro will default to high.
104  
105  - low: Minimizes latency and cost. Best for simple instruction
106    following, chat, or high-throughput applications
107  
108  - medium: (Coming soon), not supported at launch
109  
110  - high (Default): Maximizes reasoning depth. The model may take
111    significantly longer to reach a first token, but the output will be
112    more carefully reasoned.
113  
114  **Warning:** You cannot use both **thinking_level** and the
115  legacy **thinking_budget** parameter in the same request. Doing so will
116  return a 400 error.
117  
118  Media resolution
119  
120  Gemini 3 introduces granular control over multimodal vision processing
121  via the media_resolution parameter. Higher resolutions improve the
122  model's ability to read fine text or identify small details, but
123  increase token usage and latency. The media_resolution parameter
124  determines the **maximum number of tokens allocated per input image or
125  video frame.**
126  
127  You can now set the resolution
128  to media_resolution_low, media_resolution_medium,
129  or media_resolution_high per individual media part or globally
130  (via generation_config). If unspecified, the model uses optimal defaults
131  based on the media type.
132  
133  **Recommended settings**
134  
135  | Media Type | Recommended Setting | Max Tokens | Usage Guidance |
136  |----|----|----|----|
137  | Images | media_resolution_high | 1120 | Recommended for most image analysis tasks to ensure maximum quality. |
138  | PDFs | media_resolution_medium | 560 | Optimal for document understanding; quality typically saturates at medium. Increasing to high rarely improves OCR results for standard documents. |
139  | Video (General) | media_resolution_low (or media_resolution_medium) | 70 (per frame) | Note: For video, low and medium settings are treated identically (70 tokens) to optimize context usage. This is sufficient for most action recognition and description tasks. |
140  | Video (Text-heavy) | media_resolution_high | 280 (per frame) | Required only when the use case involves reading dense text (OCR) or small details within video frames. |
141  
142  **Note:** The **media_resolution** parameter maps to different token
143  counts depending on the input type. While images scale linearly
144  (**media_resolution_low**: 280, **media_resolution_medium**:
145  560, **media_resolution_high**: 1120), Video is compressed more
146  aggressively. For Video,
147  both **media_resolution_low** and **media_resolution_medium** are capped
148  at 70 tokens per frame, and **media_resolution_high** is capped at 280
149  tokens. See full
150  details [<u>here</u>](https://ai.google.dev/gemini-api/docs/media-resolution#token-counts)
151  
152  <u>[Python](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#python)[JavaScript](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#javascript)[REST](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#rest)</u>
153  
154  from google import genai
155  
156  from google.genai import types
157  
158  import base64
159  
160  \# The media_resolution parameter is currently only available in the
161  v1alpha API version.
162  
163  client = genai.Client(http_options={'api_version': 'v1alpha'})
164  
165  response = client.models.generate_content(
166  
167  model="gemini-3-pro-preview",
168  
169  contents=\[
170  
171  types.Content(
172  
173  parts=\[
174  
175  types.Part(text="What is in this image?"),
176  
177  types.Part(
178  
179  inline_data=types.Blob(
180  
181  mime_type="image/jpeg",
182  
183  data=base64.b64decode("..."),
184  
185  ),
186  
187  media_resolution={"level": "media_resolution_high"}
188  
189  )
190  
191  \]
192  
193  )
194  
195  \]
196  
197  )
198  
199  print(response.text)
200  
201  Temperature
202  
203  For Gemini 3, we strongly recommend keeping the temperature parameter at
204  its default value of 1.0.
205  
206  While previous models often benefited from tuning temperature to control
207  creativity versus determinism, Gemini 3's reasoning capabilities are
208  optimized for the default setting. Changing the temperature (setting it
209  below 1.0) may lead to unexpected behavior, such as looping or degraded
210  performance, particularly in complex mathematical or reasoning tasks.
211  
212  Thought signatures
213  
214  Gemini 3 uses [<u>Thought
215  signatures</u>](https://ai.google.dev/gemini-api/docs/thought-signatures) to
216  maintain reasoning context across API calls. These signatures are
217  encrypted representations of the model's internal thought process. To
218  ensure the model maintains its reasoning capabilities you must return
219  these signatures back to the model in your request exactly as they were
220  received:
221  
222  - **Function Calling (Strict):** The API enforces strict validation on
223    the "Current Turn". Missing signatures will result in a 400 error.
224  
225  - **Text/Chat:** Validation is not strictly enforced, but omitting
226    signatures will degrade the model's reasoning and answer quality.
227  
228  - **Image generation/editing (Strict)**: The API enforces strict
229    validation on all Model parts including a thoughtSignature. Missing
230    signatures will result in a 400 error.
231  
232  **Success:** If you use the [<u>official SDKs (Python, Node,
233  Java)</u>](https://ai.google.dev/gemini-api/docs/function-calling?example=meeting#thinking) and
234  standard chat history, Thought Signatures are handled automatically. You
235  do not need to manually manage these fields.
236  
237  Function calling (strict validation)
238  
239  When Gemini generates a functionCall, it relies on
240  the thoughtSignature to process the tool's output correctly in the next
241  turn. The "Current Turn" includes all Model (functionCall) and User
242  (functionResponse) steps that occurred since the last
243  standard **User** text message.
244  
245  - **Single Function Call:** The functionCall part contains a signature.
246    You must return it.
247  
248  - **Parallel Function Calls:** Only the first functionCall part in the
249    list will contain the signature. You must return the parts in the
250    exact order received.
251  
252  - **Multi-Step (Sequential):** If the model calls a tool, receives a
253    result, and calls *another* tool (within the same
254    turn), **both** function calls have signatures. You must
255    return **all** accumulated signatures in the history.
256  
257  Text and streaming
258  
259  For standard chat or text generation, the presence of a signature is not
260  guaranteed.
261  
262  - **Non-Streaming**: The final content part of the response may contain
263    a thoughtSignature, though it is not always present. If one is
264    returned, you should send it back to maintain best performance.
265  
266  - **Streaming**: If a signature is generated, it may arrive in a final
267    chunk that contains an empty text part. Ensure your stream parser
268    checks for signatures even if the text field is empty.
269  
270  Image generation and editing
271  
272  For gemini-3-pro-image-preview, thought signatures are critical for
273  conversational editing. When you ask the model to modify an image it
274  relies on the thoughtSignature from the previous turn to understand the
275  composition and logic of the original image.
276  
277  - **Editing:** Signatures are guaranteed on the first part after the
278    thoughts of the response (text or inlineData) and on every
279    subsequent inlineData part. You must return all of these signatures to
280    avoid errors.
281  
282  Code examples
283  
284  Multi-step Function Calling (Sequential)
285  
286  Parallel Function Calling
287  
288  Text/In-Context Reasoning (No Validation)
289  
290  Image Generation & Editing
291  
292  Migrating from other models
293  
294  If you are transferring a conversation trace from another model (e.g.,
295  Gemini 2.5) or injecting a custom function call that was not generated
296  by Gemini 3, you will not have a valid signature.
297  
298  To bypass strict validation in these specific scenarios, populate the
299  field with this specific dummy string: "thoughtSignature":
300  "context_engineering_is_the_way_to_go"
301  
302  Structured Outputs with tools
303  
304  Gemini 3 allows you to combine [<u>Structured
305  Outputs</u>](https://ai.google.dev/gemini-api/docs/structured-output) with
306  built-in tools, including [<u>Grounding with Google
307  Search</u>](https://ai.google.dev/gemini-api/docs/google-search), [<u>URL
308  Context</u>](https://ai.google.dev/gemini-api/docs/url-context),
309  and [<u>Code
310  Execution</u>](https://ai.google.dev/gemini-api/docs/code-execution).
311  
312  <u>[Python](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#python)[JavaScript](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#javascript)[REST](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#rest)</u>
313  
314  from google import genai
315  
316  from google.genai import types
317  
318  from pydantic import BaseModel, Field
319  
320  from typing import List
321  
322  class MatchResult(BaseModel):
323  
324  winner: str = Field(description="The name of the winner.")
325  
326  final_match_score: str = Field(description="The final match score.")
327  
328  scorers: List\[str\] = Field(description="The name of the scorer.")
329  
330  client = genai.Client()
331  
332  response = client.models.generate_content(
333  
334  model="gemini-3-pro-preview",
335  
336  contents="Search for all details for the latest Euro.",
337  
338  config={
339  
340  "tools": \[
341  
342  {"google_search": {}},
343  
344  {"url_context": {}}
345  
346  \],
347  
348  "response_mime_type": "application/json",
349  
350  "response_json_schema": MatchResult.model_json_schema(),
351  
352  },
353  
354  )
355  
356  result = MatchResult.model_validate_json(response.text)
357  
358  print(result)
359  
360  Image generation
361  
362  Gemini 3 Pro Image lets you generate and edit images from text prompts.
363  It uses reasoning to "think" through a prompt and can retrieve real-time
364  data—such as weather forecasts or stock charts—before using [<u>Google
365  Search</u>](https://ai.google.dev/gemini-api/docs/google-search) grounding
366  before generating high-fidelity images.
367  
368  **New & improved capabilities:**
369  
370  - **Native 4K & text rendering:** Generate sharp, legible text and
371    diagrams with native upscaling to 2K and 4K resolutions.
372  
373  - **Grounded generation:** Use the google_search tool to verify facts
374    and generate imagery based on real-world information.
375  
376  - **Conversational editing:** Multi-turn image editing by simply asking
377    for changes (e.g., "Make the background a sunset"). This workflow
378    relies on **Thought Signatures** to preserve visual context between
379    turns.
380  
381  For complete details on aspect ratios, editing workflows, and
382  configuration options, see the [<u>Image Generation
383  guide</u>](https://ai.google.dev/gemini-api/docs/image-generation).
384  
385  <u>[Python](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#python)[JavaScript](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#javascript)[REST](https://ai.google.dev/gemini-api/docs/gemini-3?thinking=high#rest)</u>
386  
387  from google import genai
388  
389  from google.genai import types
390  
391  client = genai.Client()
392  
393  response = client.models.generate_content(
394  
395  model="gemini-3-pro-image-preview",
396  
397  contents="Generate an infographic of the current weather in Tokyo.",
398  
399  config=types.GenerateContentConfig(
400  
401  tools=\[{"google_search": {}}\],
402  
403  image_config=types.ImageConfig(
404  
405  aspect_ratio="16:9",
406  
407  image_size="4K"
408  
409  )
410  
411  )
412  
413  )
414  
415  image_parts = \[part for part in response.parts if part.inline_data\]
416  
417  if image_parts:
418  
419  image = image_parts\[0\].as_image()
420  
421  image.save('weather_tokyo.png')
422  
423  image.show()
424  
425  **Example Response**
426  
427  <img src="media/image2.jpeg" style="width:6.5in;height:3.62708in"
428  alt="Weather Tokyo" />
429  
430  Migrating from Gemini 2.5
431  
432  Gemini 3 is our most capable model family to date and offers a stepwise
433  improvement over Gemini 2.5 Pro. When migrating, consider the following:
434  
435  - **Thinking:** If you were previously using complex prompt engineering
436    (like Chain-of-thought) to force Gemini 2.5 to reason, try Gemini 3
437    with thinking_level: "high" and simplified prompts.
438  
439  - **Temperature settings:** If your existing code explicitly sets
440    temperature (especially to low values for deterministic outputs), we
441    recommend removing this parameter and using the Gemini 3 default of
442    1.0 to avoid potential looping issues or performance degradation on
443    complex tasks.
444  
445  - **PDF & document understanding:** Default OCR resolution for PDFs has
446    changed. If you relied on specific behavior for dense document
447    parsing, test the new media_resolution_high setting to ensure
448    continued accuracy.
449  
450  - **Token consumption:** Migrating to Gemini 3 Pro defaults
451    may **increase** token usage for PDFs but **decrease** token usage for
452    video. If requests now exceed the context window due to higher default
453    resolutions, we recommend explicitly reducing the media resolution.
454  
455  - **Image segmentation:** Image segmentation capabilities (returning
456    pixel-level masks for objects) are not supported in Gemini 3 Pro. For
457    workloads requiring native image segmentation, we recommend continuing
458    to utilize Gemini 2.5 Flash with thinking turned off or [<u>Gemini
459    Robotics-ER
460    1.5</u>](https://ai.google.dev/gemini-api/docs/robotics-overview).
461  
462  OpenAI compatibility
463  
464  For users utilizing the OpenAI compatibility layer, standard parameters
465  are automatically mapped to Gemini equivalents:
466  
467  - reasoning_effort (OAI) maps to thinking_level (Gemini). Note
468    that reasoning_effort medium maps to thinking_level high.
469  
470  Prompting best practices
471  
472  Gemini 3 is a reasoning model, which changes how you should prompt.
473  
474  - **Precise instructions:** Be concise in your input prompts. Gemini 3
475    responds best to direct, clear instructions. It may over-analyze
476    verbose or overly complex prompt engineering techniques used for older
477    models.
478  
479  - **Output verbosity:** By default, Gemini 3 is less verbose and prefers
480    providing direct, efficient answers. If your use case requires a more
481    conversational or "chatty" persona, you must explicitly steer the
482    model in the prompt (e.g., "Explain this as a friendly, talkative
483    assistant").
484  
485  - **Context management:** When working with large datasets (e.g., entire
486    books, codebases, or long videos), place your specific instructions or
487    questions at the end of the prompt, after the data context. Anchor the
488    model's reasoning to the provided data by starting your question with
489    a phrase like, "Based on the information above...".
490  
491  Learn more about prompt design strategies in the [<u>prompt engineering
492  guide</u>](https://ai.google.dev/gemini-api/docs/prompting-strategies).
493  
494  FAQ
495  
496  1.  **What is the knowledge cutoff for Gemini 3 Pro?** Gemini 3 has a
497      knowledge cutoff of January 2025. For more recent information, use
498      the [<u>Search
499      Grounding</u>](https://ai.google.dev/gemini-api/docs/google-search) tool.
500  
501  2.  **What are the context window limits?** Gemini 3 Pro supports a 1
502      million token input context window and up to 64k tokens of output.
503  
504  3.  **Is there a free tier for Gemini 3 Pro?** You can try the model for
505      free in Google AI Studio, but currently, there is no free tier
506      available for gemini-3-pro-preview in the Gemini API.
507  
508  4.  **Will my old thinking_budget code still
509      work?** Yes, thinking_budget is still supported for backward
510      compatibility, but we recommend migrating to thinking_level for more
511      predictable performance. Do not use both in the same request.
512  
513  5.  **Does Gemini 3 support the Batch API?** Yes, Gemini 3 supports
514      the [<u>Batch
515      API.</u>](https://ai.google.dev/gemini-api/docs/batch-api)
516  
517  6.  **Is Context Caching supported?** Yes, [<u>Context
518      Caching</u>](https://ai.google.dev/gemini-api/docs/caching?lang=python) is
519      supported for Gemini 3. The minimum token count required to initiate
520      caching is 2,048 tokens.
521  
522  7.  **Which tools are supported in Gemini 3?** Gemini 3
523      supports [<u>Google
524      Search</u>](https://ai.google.dev/gemini-api/docs/google-search), [<u>File
525      Search</u>](https://ai.google.dev/gemini-api/docs/file-search), [<u>Code
526      Execution</u>](https://ai.google.dev/gemini-api/docs/code-execution),
527      and [<u>URL
528      Context</u>](https://ai.google.dev/gemini-api/docs/url-context). It
529      also supports standard [<u>Function
530      Calling</u>](https://ai.google.dev/gemini-api/docs/function-calling?example=meeting) for
531      your own custom tools. Please note that [<u>Google
532      Maps</u>](https://ai.google.dev/gemini-api/docs/maps-grounding) and [<u>Computer
533      Use</u>](https://ai.google.dev/gemini-api/docs/computer-use) are
534      currently not supported.
535  
536  Next steps
537  
538  - Get started with the [<u>Gemini 3
539    Cookbook</u>](https://colab.research.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_started.ipynb#templateParams=%7B%22MODEL_ID%22%3A+%22gemini-3-pro-preview%22%7D)
540  
541  - Check the dedicated Cookbook guide on [<u>thinking
542    levels</u>](https://colab.research.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Get_started_thinking_REST.ipynb#gemini3) and
543    how to migrate from thinking budget to thinking levels.
```
