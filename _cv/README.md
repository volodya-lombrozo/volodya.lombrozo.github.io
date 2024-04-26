# CV

Here is my CV. It is written in LaTeX. You can find 3 different versions of my
CV in the `_cv` directory:

- `cv.pdf`: The full version of my CV.
- `cv-short.pdf`: A short version of my CV.
- `cv-1page.pdf`: A one-page version of my CV.

## How to build

To build the CV, you need to
install [latexmk](https://mg.readthedocs.io/latexmk.html),
[texsc](https://github.com/yegor256/texsc),
and [texqc](https://github.com/yegor256/texqc). Also you need to
install [aspell](http://aspell.net). To build the CV, run the
following command:

```bash
make all
```

To clean the build files, run the following command:

```bash
make clean
```