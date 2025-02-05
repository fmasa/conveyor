# Download Conveyor

[ :material-download-circle: Download for your OS](https://downloads.hydraulic.dev/conveyor/download.html){ .md-button .md-button--primary }

After downloading:

* **On macOS**, open the app and click "Add to shell path" so you can use the `conveyor` command from the terminal. You'll need to re-open terminal tabs to apply the `$PATH` change.
* **On Windows**, make sure you are using [Windows Terminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701). The legacy default Windows console has some rendering bugs. Conveyor will appear in your `%PATH%` automatically so you don't need to restart any terminals.  

Now run `conveyor --help` to check it's ready to go and move on to the next section.

!!! note "Automatic updates"
    Conveyor is packaged with itself, but automatic updates are switched off for Windows and macOS to give you full control. To update Conveyor, run it from the GUI and then click "Check for updates". On Debian you can control updates in the usual manner and for other distros, the tarball never updates itself so you'll need to re-download it.

## Downloading in build scripts

You can download specific versions of Conveyor by simply putting the URLs you find on the download page into your build scripts. Please observe the following:

1. Ensure your CI system is caching the download. If we observe IP addresses redownloading over and over they'll be throttled or blocked. Consider mirroring the download on your own servers if you aren't sure how to set up caching.
2. If you want to use the Windows version you can just grab the MSIX file and unzip it.
3. You can use command line flags or environment variables to agree to the license. You'll be given instructions on what to do the first time you manually agree.

## Next steps

1. Please read the [release notes](release-notes.md) and [known issues list](known-issues.md)!
1. Learn [what Conveyor produces](outputs.md).
1. Take [the tutorial](tutorial/new.md).
1. Learn more about how to [run the tool](running.md).
1. Learn [how to customize the results](configs/index.md).
