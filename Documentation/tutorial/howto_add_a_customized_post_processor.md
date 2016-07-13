How-to: Add a customized post-processor
====================================

We would have the ability to process output files by adding a customized post-processor.
In DocFX, the index file for full-text-search is generated by one post-processor named `ExtractSearchIndex`.
In this topic, we will show how to add a customized post-processor.

## Step0: Preparation

* Create a new C# class library project in `Visual Studio`.
* Add nuget packages:
    * `System.Collections.Immutable` with version 1.1.37
    * `Microsoft.Composition` with version 1.0.27
* Add `Microsoft.DocAsCode.Plugins`
If you are building DocFX from source code, add reference to the project.
Otherwise, add nuget package `Microsoft.DocAsCode.Plugins` with the same version of DocFX.

## Step1: Create a new class (MyProcessor.cs) with following code:

```csharp
[Export(nameof(MyProcessor), typeof(IPostProcessor))]
public class MyProcessor : IPostProcessor
{
    // TODO: implements IPostProcessor
}
```

## Step2: Update global metadata

```csharp
public ImmutableDictionary<string, object> PrepareMetadata(ImmutableDictionary<string, object> metadata)
{
    // TODO: add/remove/update property from global metadata
    return metadata;
}
```

In this method, we can update the global metadata before building all the files declared in `docfx.json`. Otherwise, you can just return the metadata from parameter if you don't need to change global metadata.

Take `ExtractSearchIndex` for example, we add `"_enableSearch": true` in global metadata. Then, the default template would know it should load search box in navbar.

## Step3: Process all the files generated by DocFX

```csharp
    public Manifest Process(Manifest manifest, string outputFolder)
    {
        // TODO: add/remove/update all the files included in manifest
        return manifest;
    }
```

The input of this method `manifest` contains a list of all files to process, and `outputFolder` specifies the output folder where our static website will be placed. We can implement customized operations here to process all files generated by DocFX.

Take `ExtractSearchIndex` for example again, we traverse all HTML files, extract key words from these HTML files and save a file named `index.json` under the `outputFolder`. At last, we return the manifest which is not modified.

## Step4: Build your project and copy the output dll files to:

* Global: the folder with name `Plugins` under DocFX.exe
* Non-global: the folder with name `Plugins` under a template folder, then run `DocFX build` command with parameter `-t {template}`.

    *Hint*: DocFX can merge templates, so we can specify multiple template folders as `DocFX build -t {templateForRender},{templateForPlugins}`. Each of template folder should have a subfolder named `Plugins` with exported assemblies.

## Step5: Add your post processor in `docfx.json`

In this step, we need to enable the processor by adding its name in `docfx.json`. There is an example as following:

```json
{
  "build": {
    ...
    "postprocessors": ["OutputPDF", "BeautifyHTML", "OutputPDF"]
  }
}
```

As you can see, the `postprocessors` is an array, which means it could have multiple processors.
Needs to be pointed out is that the order of `postprocessors` written in `docfx.json` is also the order to process output files.
In above example, DocFX will run `OutputPDF` first, then `BeautifyHTML`, and then `OutputPDF` again.