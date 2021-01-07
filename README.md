# Newsfeed - MVP


## Getting Started


### Prerequisites

* Docker: download can be found [here](https://www.docker.com/get-started)
* [ACT](https://github.com/nektos/act) - Run Github actions locally
    * Mac: (Assuming Homebrew is installed): `brew install act`
    * Win 10: Chocolatey currently not working, download the [latest release](https://github.com/nektos/act/releases/latest) and place the binary in your `PATH`.
* Add the following SSM parameter `/local/newsfeed/newsfeed-token` and set it to the appropriate token.  Since this is considered a secret, it shouldn't be checked in to source control.
