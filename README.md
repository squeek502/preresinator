# preresinator

An experimental fork of [arocc](https://github.com/Vexu/arocc) specifically geared towards preprocessing Windows resource scripts (`.rc` files) for [resinator](https://github.com/squeek502/resinator).

## Why a fork?

The ultimate goal is to use this as a preprocessor for [resinator](https://github.com/squeek502/resinator) in order to support UTF-16 encoded `.rc` files (see [#5](https://github.com/squeek502/resinator/issues/5)). This ends up being nontrivial for a few reasons:

- Resource scripts have a special `#pragma code_page` preprocessor directive which means that there can be multiple text encodings *within* a given `.rc` file (each line could have a separate encoding in theory).
- `#pragma code_page` has [a strange quirk](https://squeek502.github.io/resinator/windows/input-and-output-code-pages.html) which means that the precise number and ordering of `#pragma code_page` directives matters.

Together, this means that it's not as simple as 'parse UTF-16 encoded files and output them as UTF-8', since in order to let the resource compiler know that the formerly-UTF-16 encoded lines should be treated as UTF-8 without introducing any new features, `#pragma code_page` directives would need to be injected into the output which can change how the file is parsed.

So, the possible solutions as I currently see them:

- Parse UTF-16 and output UTF-8, but have some special non-standard way of communicating the code page for those lines. I don't particularly like this solution.
- Make the preprocessor handle *all* `#pragma code_page` directives by 'consuming' them and have it output *only* UTF-8 regardless of the input encoding. This means that the preprocessor would need to e.g. parse strings as the input encoding in order to output them correctly (not something that a C preprocessor would normally need to do).

I could be wrong about the above, and I don't currently feel like I have the full picture, so experimenting and seeing what might actually work is the necessary first step.

### Other minor reasons

- Only needs to be a preprocessor, so all of the non-preprocessor code can be removed from this fork
- According to the `rc.exe` documentation, [only certain preprocessor directives](https://learn.microsoft.com/en-us/windows/win32/menurc/preprocessor-directives) / [macros](https://learn.microsoft.com/en-us/windows/win32/menurc/predefined-macros) / [operators](https://learn.microsoft.com/en-us/windows/win32/menurc/preprocessor-operators) / [pragmas](https://learn.microsoft.com/en-us/windows/win32/menurc/pragma-directives) need to be supported. This may not end up being fully true, but in theory that might allow the preprocessor to be simpler than a fully compliant C compiler needs.
- [Long-term] One idea I want to try is including a compressed set of MinGW headers bundled in the `resinator` exe, and then make the preprocessor be able to use them as-needed without needing to extract the full set of headers on disk. This would allow `resinator` to be an all-in-one/zero-dependency/out-of-the-box resource compiler for any host platform (right now, `resinator` relies on a Zig install for similar functionality).
