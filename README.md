# TypExt
A Typescript factory of ExtJS classes

The goal of this project is to provide a brige between TypeScript and the ExtJS framework.

## ExtJS and TypeScript: a complicated story...

It is well known that TypeScript (TS) and ExtJS don't play well together because they use incompatible class systems:

>https://blorkfish.wordpress.com/2013/01/28/using-extjs-with-typescript/ <br/>
>Unfortunately, TypeScript and ExtJs do not seem to work too well together.  <br/>
>This incompatibility is mainly due to the differences in object creation between the two approaches. <br/>
>Where ExtJs uses config blocks and anonymous methods for object creation, TypeScript uses the closure pattern to bring an easier way to build object-oriented javascript. <br/>
>Unfortunately these two approaches seem to be at odds with each other. <br/>

Despite these limitations, and also the low interest from Sencha in making progress on this issue, some volunteer developers worked on  setting up available ExtJS-TS definitions. <br/> 
To generate TS definitions in an automated way, these developers got the idea to use the ExtJS documentation as a data source.
The strategy consists in executing the Sencha *JSDuck* documentation tool on the ExtJS sources in order to generate corresponding JSON files.
These files are then parsed by a dedicated converter which will generate the precious TS definitions.

Here is the list of projects providing TypeScript definitions for ExtJS (and for some of them the corresponding "JSDuck to TS" converter):
* https://github.com/zz9pa/extjsTypescript - ExtJS 4.1.1 - Last update Dec 13, 2012 <br/> 
This work by Mike Aubury (zz9pa) is presented in the blogpost cited just above.
The converter is written in C# and to my knowlege this was the first attempt to provide a set of TS definitions for ExtJS.

* https://github.com/brian428/ext-typescript-generator - ExtJS 4.2.1 - Last update Dec 13, 2013 <br/> 
Another converter (code written in Groovy) developed by Brian Kotek.
I forked this repository for my own needs.

* https://github.com/Dretch/typescript-declarations-for-ext - ExtJS 4.2.1 & 5.1.0 - Last update Apr 30, 2015 <br/> 
An other converter (code written in TypeScript) developed by Gareth Smith.
To be used in conjuction with fabioparra's patched Typescript compiler (see below).

* https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/extjs - ExtJS 4.2.1 - Last update Mar 17, 2016 <br/>
The ExtJS definitions available from the well-known DefinitelyTyped repository.
Note that the definitions seem to be those generated by Brian Kotek.

* https://github.com/david-bouyssie/TypExt/tree/master/typescripts - ExtJS 5.1.1  - Last update Jul 27, 2016 <br/>
This is my personnal contribution to existing TS definitions.
As mentioned above the converter has been forked from brian428's repository.
Note that I made changes to this converter to make it compatible with ExtJS 5.1.1 but also to generate files relative to the TypExt project.

* https://github.com/thanhptr/extts - ExtJS 4.2.1, 5.1.1 & 6.0.2 (modern/classic) - Last update Aug 15, 2016 <br/>
This repo maintained by Thanh Pham is the most recent source of TypeScript definitions.

While the availability of such TS definitions allows a partial comprehension of ExtJS API by the TypeScript compiler,
it does not offer a fully compile-time checking system when dealing with ExtJS classes instanciation.

This is why some other devs worked on a forked version of TS compiler to emit JavaScript code in an ExtJS class style:

* https://typescript4extjs.codeplex.com/ - Last update Jul 16, 2014 <br/>
Abandonned project (version 0.0.1 alpha), seems to be a proof-of-concept only.

* https://github.com/fabioparra/TypeScriptExtJSEmitter - TypeScript - 1.0.3 - Last update Jan 31, 2015 <br/>
First fabioparra's fork of the TypeScript compiler.
I think it was also a proof-of-concept.

* https://github.com/fabioparra/TypeScript - TypeScript - 1.8 - Last update Apr 8, 2016 <br/>
The current fabioparra's fork of the TypeScript compiler. Very active project.

## Motivation for an alternative solution

So you may wonder why develop another solution if all the great definitions/tools mentionned above are available ?
Because none of them may satisfy your needs.

In my opinion the main drawback of the *Patched Compiler* approach is that it complicates the integration with IDE supporting the TS language.
For instance there is no documentation regarding the compatibility of this solution with existing Eclipse TS plugins.
Moreover it is not sure that the development of patch will be maintained for future versions of TypeScript.
If you have a look at the current repo documentation, it says that it is compatible only with TS up to 1.8 and ExtJS up to 5.x.
What about TS 2.0 and ExtJS 6.x ?

The second drawback is relative to Javascript performance.
The TS compiler does a lot of work to generate JS code that will be as fast as possible.
Emitting JS code using the ExtJS style means that a lot of things will occur at runtime and this may not be what you wish to obtain for some parts of your code.

So, are we stuck ?

I think there is another way that has not been explored so far: use TS static methods as a factory of ExtJS objects.

## TypExt: compile-time checking of your ExtJS classes instantiation

The main idea of TypExt is to provide a set of static methods replacing the ExtJS constructors.
It thus becomes possible to instantiate ExtJS classes while using the genuine TS compiler.

TO BE DETAILED

ExtJS 5.1.1 only is currently supported, but I plan to extend the usage of TypExt with other ExtJS versions.

## TypExt addons

TypExt major addons are shortly described below:

### 1. TypExt.addon.data.IModelField



```javascript
module path.to.my.module {

  import IModelField = TypExt.addon.data.IModelField;

  export interface IFile {
    id: string;
    path: string;
    name: string;
    size: number;
    lastmodified: number;
    extension: string;
  }

  export class FileModelFactory extends TypExt.addon.data.AbstractModelFactory implements IFile {
    getClassName() { return "PWD.module.dse.model.File" }

    id: any = <IModelField>{ name: 'id', type: 'string' };
    path: any = <IModelField>{ name: 'path', type: 'string' };
    name: any = <IModelField>{ name: 'name', type: 'string' };
    size: any = <IModelField>{ name: 'size', type: 'int' };
    lastmodified: any = <IModelField>{ name: 'lastmodified', type: 'int' };
    extension: any = <IModelField>{ name: 'extension', type: 'string' };
  }

  new FileModelFactory().defineModel();
}
```
Note: you need to force the type 'any' to avoid the following error: 
```javascript
Type 'IModelField' is not assignable to type 'number'.
```

You can also use ExtJS automatic typing (recommanded for strings only):
```javascript
...
  export class FileModelFactory extends TypExt.addon.data.AbstractModelFactory implements IFile {
    getClassName() { return "PWD.module.dse.model.File" }
    id = 'id';
    path = 'path';
    name = 'name';
    size: any = <IModelField>{ name: 'size', type: 'int' }; // prefer to define it explicitly for non-string types
    lastmodified: any = <IModelField>{ name: 'lastmodified', type: 'int' };
    extension = 'extension';
  }
...
```

### 2. TypExt.addon.Ajax

TypExt provides shortcut methods *httpPost* and *httpGet* for HTTP calls, for example:
```javascript
TypExt.addon.Ajax.httpPost({
  url: "www.my.url",
  params: {...},
  callback: function(options: TypExt.ajax.IRequestConfig, success: boolean, response: XMLHttpRequest) {
    if (success) ... else ...
  }
});
```

### 3. TypExt.addon.panel.IPanel

The main advantage of this addon panel over a classical one is that it provides the *createAndExtend()* method. It will return the panel as an *IExtendedContainer* (also a TypExt addon), much more powerfull.
For example, and that was the initial motivation for creating it, assessing a panel children in javascript can be very tricky. The addon panel provides an utility method to get them more easily:
```javascript
const addonPanel = TypExt.addon.Panel.createAndExtend({
  config...
});
const embeddedPanels: TypExt.addon.panel.IPanel[] = addonPanel.getItems();
```


## Caveats

TypExt doesn't solve entirely the TS/ExtJS integration.
It just provides more safety regarding the instantiation of ExtJS classes.

### 1. ExtJS inheritance

If you want to extend ExtJS classes using the ExtJS syntax (Ext.define) you will have to maintain a separate TS interface corresponding to the config object that will be passed to your class constructor.

This issue is well described in this Stackoverflow answer:
>http://stackoverflow.com/questions/24671996/typescript-definitions-for-extjs-5 <br/>
>The problem is that there are two type systems in play. <br/>
>One is the design-time type system enforced by the TypeScript compiler. <br/>
>The other is the run-time type system defined by ExtJS. <br/>
>These two type systems are very similar in regard to the way they are implemented, yet they each have a distinct syntax, and therefore have to be specified seperately. <br/>
>Much of the value of a static type system is lost if it two copies of every interface have to be maintained. <br/>

*Suggested workarounds*: 

1. Do not use ExtJS inheritance when possible.
Define as much TS classes as possible.
This way you will decrease the need of maintaining both a TS interface and an ExtJS Ext.define block.

2. When defining stores or models, use the dedicate TypExt addon (see above).
It allows you to define a model using the TS syntax while generating a model compatible with the ExtJS framework.

### 2. ExtJS dynamic class loading

Ext.create is responsible of lazy instantiating ExtJS classes.
If Ext.Loader is enabled and the class has not been defined yet, it will attempt to load the class for you via synchronous loading.
The loader will look for a JS file at the given path following the Ext.Loader convention.
Note that the Ext.Loader is fully configurable:
http://docs.sencha.com/extjs/6.2.0/modern/Ext.Loader.html#property-setConfig
http://extendedjs.blogspot.fr/2012/12/extjs-4classesdynamic-loading.html

The problem is that you will have to:
* configure your TS compiler to generate one JS file per TS file.
* place a Ext.define() statement at the bottom of your TS file that will create an ExtJS class that will be detected by the Ext.Loader to success the class loading process.

*Suggested workaround*: do not use the ExtJS loader anymore and genarate a single JS file.
You will loose dynamic ExtJS class loading but in most cases you won't affect the application performance.

