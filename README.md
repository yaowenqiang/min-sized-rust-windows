# Minimum Binary Size Windows
An example of how small a rust binary can get on windows 10. This isn't something meant to be used in production more, of a challenge.  I'm in no ways an expert and 
[I have seen windows binaries get smaller on windows](https://github.com/pts/pts-tinype). If
you can go smaller let me know how you did it :grin:

### Results
`1k` :sunglasses:

```powershell
❯ cargo run --release
Hello World!

❯ cargo build --release && (Get-Item ".\target\release\min-sized-rust-windows.exe").Length
    Finished release [optimized] target(s) in 0.02s
1024
```

### Strategies
I'm excluding basic strategies here such as enabling lto and setting `opt-level = 'z'`.

* [`no_std`](https://github.com/johnthagen/min-sized-rust#removing-libstd-with-no_std)
* [`no_main`](https://github.com/johnthagen/min-sized-rust#remove-corefmt-with-no_main-and-careful-usage-of-libstd)
* Merge `.rdata` and `.pdata` sections into `.text` section linker flag.
    * Using the LINK.exe [`/MERGE`](https://docs.microsoft.com/en-us/cpp/build/reference/merge-combine-sections?view=msvc-160)     
      flag found at the bottom of `main.rs`.
    * Windows 10 section size appears to be locked at `1024`, so to avoid anything larger
    we merge everything into the `.text` section.
* `PEB` export walker
    * Piggy backing off of the previous findings we have a new issue.  When importing
    `kernel32.lib` we are stuck with an extra `1024`b `.idata` section.  Unfortunately
    `LINK.exe` really doesn't like it when you try to merge `.idata => .text`.  Since 
    `kernel32.dll` is accessible by default via the `PEB` I opted to obtain a reference to
    it and search the export directory for `GetStdHandle` and `WriteFile`.
      
### References
* https://github.com/johnthagen/min-sized-rust
* http://www.catch22.net/tuts/win32/reducing-executable-size
* https://github.com/pts/pts-tinype
