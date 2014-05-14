# Private-Local-Cocoapods-Guide

Guide to setup local and thus private cocoa pods without a git server but full feature set of SCM.

## Structure your private snippets

First you need to create a folder and a  git repository for each code snippet that you want to keep private and put them all into one folder. Your hirachy shoud look something like this:

	+-- ~/Projects/Cocoapods/Local Cocoapods/Sources
	| +-- NSView+SubviewExtension
	  | +-- .git
	  | +-- NSView+SubviewExtension.h
	  | +-- NSView+SubviewExtension.m
	| +-- NSString+Extensions
	  | +-- .git
	  | +-- NSString+Extensions.h
	  | +-- NSString+Extensions.m
		
The structure may differ, you have to specify your Podspec files correspondently.
These repos contains your sources and should be managed as described [here](http://guides.cocoapods.org/making/making-a-cocoapod.html).

## Create your private Podspecs

An appropriate Podspec file could be like this:

```
Pod::Spec.new do |s|
    s.name = 'NSView+SubviewExtensions'
    s.version = '1.0.0'
    s.platform = :osx, '10.6'
    s.authors = {'Your name' => 'your@email.com'}
    s.license = {:type => 'MIT or whatever'}
    s.homepage = 'http://no-homepage.com'
    s.summary = 'Access all subviews of NSView recursively.'
    s.source = {:git => 'git://127.0.0.1/NSView+SubviewExtensions', :tag => 'v1.0.0'}
    s.source_files = '*.{h,m}'
    s.requires_arc = false
end
```
The git-source attrbute is the place where the "magic" happens.

Next you need to create a local podspec repo and add it to Cocoapods. [Here](http://guides.cocoapods.org/making/private-cocoapods.html) is described how you do that.

It could look someting like this:

	+-- ~/Projects/Cocoapods/Local Cocoapods/Podspecs
	| +-- .git
	| +-- NSView+SubviewExtension
	  | +-- 1.0.0
	    | +-- NSView+SubviewExtension.podspec
	  | +-- 1.0.1
	    | +-- NSView+SubviewExtension.podspec
	| +-- NSString+Extensions
	  | +-- 1.0.0
	    | +-- NSString+Extensions.h
	    
With the following command you add the local specs repo to cocoapods:

	pod repo add LocalSpecs "~/Projects/Local Cocoapods/Podspecs"
	
Everytime you create a new version of one of your local cocoapods and add a correspondent podspec file to your local specs repo you have to commit the changes and tell Cocoapods that the repo has updated:

	pod repo update LocalSpecs

## Local git-daemon or rather server

The next step is to create a local git daemon which is accessable via `127.0.0.1`. You can do this with the following command:

	git daemon --base-path="Users/[yourUserName]/Projects/Local Cocoapods/Sources" --export-all --reuseaddr --informative-errors --verbose

I created an alias for that command because you will need it everytime Cocoapods needs to access a local cocoapod. The running git-daemon "looks" like this:

![Alternativtext](GitDaemon.png "")

As long as you do not close this window the git daemon or rather server is running. Close this window when Cocoapods has finished installing pods.

## Using private local Cocoapods

In your podfile you can specify the local cocoapod like a global cocoapods

``` ruby
pod 'NSView+SubviewExtension', '1.0.1'
```

Start the git-daemon before calling `pod install`.