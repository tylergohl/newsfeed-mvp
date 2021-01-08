# Newsfeed - MVP


## Getting Started


### **Prerequisites**

* AWS Account
* Docker: download can be found [here](https://www.docker.com/get-started)
* [ACT](https://github.com/nektos/act) - For running Github actions locally
    * Mac: (Assuming Homebrew is installed): `brew install act`
    * Win 10: Chocolatey currently not working, download the [latest release](https://github.com/nektos/act/releases/latest) and place the binary in your `PATH`.


### **Build / Deploy Environment**

* Make sure the following environment variables have been set:
    * `AWS_ACCESS_KEY_ID`
    * `AWS_SECRET_ACCESS_KEY`
    * `AWS_DEFAULT_REGION`
    * `NEWSFEED_TOKEN` (this should be set to the value that the code expects for this secret)

* To build the environment run: `act -s AWS_ACCESS_KEY_ID -s AWS_SECRET_ACCESS_KEY -s AWS_DEFAULT_REGION -s NEWSFEED_TOKEN`