---
title: Test a .NET class library using Visual Studio Code
description: Learn how to use Visual Studio Code and run a unit test project for a .NET class library.
ms.date: 06/14/2024
---
# Tutorial: Test a .NET class library using Visual Studio Code

This tutorial shows how to automate unit testing by adding a test project to a solution.

## Prerequisites

- This tutorial works with the solution that you create in [Create a .NET class library using Visual Studio Code](library-with-visual-studio-code.md).

## Create a unit test project

Unit tests provide automated software testing during your development and publishing. The testing framework that you use in this tutorial is MSTest. [MSTest](https://github.com/Microsoft/testfx-docs) is one of three test frameworks you can choose from. The others are [xUnit](https://xunit.net/) and [nUnit](https://nunit.org/).

1. Start Visual Studio Code.

1. Open the `ClassLibraryProjects` solution you created in [Create a .NET class library using Visual Studio Code](library-with-visual-studio-code.md).

1. From **Solution Explorer**, select **New Project**, or from the Command Palette select **.NET: New Project**.

1. Select **MSTest Test Project**, name it "StringLibraryTest", select the default directory, and select **Create Project**.

   The project template creates a *UnitTest1.cs* file with the following code:

   ```csharp
   namespace StringLibraryTest;

   [TestClass]
   public class UnitTest1
   {
       [TestMethod]
       public void TestMethod1()
       {
       }
   }
   ```

   The source code created by the unit test template does the following:

   - It applies the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.TestClassAttribute> attribute to the `UnitTest1` class.
   - It applies the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.TestMethodAttribute> attribute to define `TestMethod1`.
   - It imports the <xref:Microsoft.VisualStudio.TestTools.UnitTesting?displayProperty=nameWithType> namespace, which contains the types used for unit testing. The namespace is imported via a `global using` directive in *GlobalUsings.cs*.

   Each method tagged with [[TestMethod]](xref:Microsoft.VisualStudio.TestTools.UnitTesting.TestMethodAttribute) in a test class tagged with [[TestClass]](xref:Microsoft.VisualStudio.TestTools.UnitTesting.TestClassAttribute) is run automatically when the unit test is invoked.

## Add a project reference

For the test project to work with the `StringLibrary` class, add a reference in the `StringLibraryTest` project to the `StringLibrary` project.

1. From **Solution Explorer** right click on the 'StringLibraryTest' Project and select **Add Project Reference**.

1. Select "StringLibrary".

## Add and run unit test methods

When Visual Studio invokes a unit test, it runs each method that is marked with the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.TestMethodAttribute> attribute in a class that is marked with the  <xref:Microsoft.VisualStudio.TestTools.UnitTesting.TestClassAttribute> attribute. A test method ends when the first failure is found or when all tests contained in the method have succeeded.

The most common tests call members of the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.Assert> class. Many assert methods include at least two parameters, one of which is the expected test result and the other of which is the actual test result. Some of the `Assert` class's most frequently called methods are shown in the following table:

| Assert methods     | Function |
| ------------------ | -------- |
| `Assert.AreEqual`  | Verifies that two values or objects are equal. The assert fails if the values or objects aren't equal. |
| `Assert.AreSame`   | Verifies that two object variables refer to the same object. The assert fails if the variables refer to different objects. |
| `Assert.IsFalse`   | Verifies that a condition is `false`. The assert fails if the condition is `true`. |
| `Assert.IsNotNull` | Verifies that an object isn't `null`. The assert fails if the object is `null`. |

You can also use the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.Assert.Throws*?displayProperty=nameWithType> method in a test method to indicate the type of exception it's expected to throw. The test fails if the specified exception isn't thrown.

In testing the `StringLibrary.StartsWithUpper` method, you want to provide a number of strings that begin with an uppercase character. You expect the method to return `true` in these cases, so you can call the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.Assert.IsTrue%2A?displayProperty=nameWithType> method. Similarly, you want to provide a number of strings that begin with something other than an uppercase character. You expect the method to return `false` in these cases, so you can call the <xref:Microsoft.VisualStudio.TestTools.UnitTesting.Assert.IsFalse%2A?displayProperty=nameWithType> method.

Since your library method handles strings, you also want to make sure that it successfully handles an [empty string (`String.Empty`)](xref:System.String.Empty) and a `null` string. An empty string is one that has no characters and whose <xref:System.String.Length> is 0. A `null` string is one that hasn't been initialized. You can call `StartsWithUpper` directly as a static method and pass a single <xref:System.String> argument. Or you can call `StartsWithUpper` as an extension method on a `string` variable assigned to `null`.

You'll define three methods, each of which calls an <xref:Microsoft.VisualStudio.TestTools.UnitTesting.Assert> method for each element in a string array. You'll call a method overload that lets you specify an error message to be displayed in case of test failure. The message identifies the string that caused the failure.

To create the test methods:

1. Open *StringLibraryTest/UnitTest1.cs* and replace all of the code with the following code.

   :::code language="csharp" source="./snippets/library-with-visual-studio/csharp/StringLibraryTest/UnitTest1.cs":::

   The test of uppercase characters in the `TestStartsWithUpper` method includes the Greek capital letter alpha (U+0391) and the Cyrillic capital letter EM (U+041C). The test of lowercase characters in the `TestDoesNotStartWithUpper` method includes the Greek small letter alpha (U+03B1) and the Cyrillic small letter Ghe (U+0433).

1. Save your changes.

## Build and Run your Tests

1. In **Solution Explorer**, right-click the solution and select **Build** or from the Command Palette, select **.NET: Build**.

1. Select the **Testing** window, select **Run Tests** or from the Command Palette, select **Test: Run all Tests**.

   :::image type="content" source="media/testing-library-with-visual-studio-code/testingScreenshot.png" alt-text="Visual Studio Code Test Explorer":::

## Handle test failures

If you're doing test-driven development (TDD), you write tests first and they fail the first time you run them. Then you add code to the app that makes the test succeed. For this tutorial, you created the test after writing the app code that it validates, so you haven't seen the test fail. To validate that a test fails when you expect it to fail, add an invalid value to the test input.

1. Modify the `words` array in the `TestDoesNotStartWithUpper` method to include the string "Error".

   ```csharp
   string[] words = { "alphabet", "Error", "zebra", "abc", "αυτοκινητοβιομηχανία", "государство",
                      "1234", ".", ";", " " };
   ```

1. Run the tests by clicking on the green error next to the test in the editor.

   The output shows that the test fails, and it provides an error message for the failed test: "Assert.IsFalse failed. Expected for 'Error': false; actual: True". Because of the failure, no strings in the array after "Error" were tested.

      :::image type="content" source="media/testing-library-with-visual-studio-code/failedTest.png" alt-text="Visual Studio Code Failed Test":::

1. Remove the string "Error" that you added in step.

1. Rerun the test and the tests pass.

## Test the Release version of the library

Now that the tests have all passed when running the Debug build of the library, run the tests an additional time against the Release build of the library. A number of factors, including compiler optimizations, can sometimes produce different behavior between Debug and Release builds.

1. Run the tests with the Release build configuration:

   ```dotnetcli
   dotnet test StringLibraryTest/StringLibraryTest.csproj --configuration Release
   ```

   The tests pass.

## Debug tests

If you're using Visual Studio Code as your IDE, you can use the same process shown in [Debug a .NET console application using Visual Studio Code](debugging-with-visual-studio-code.md) to debug code using your unit test project. Instead of starting the *ShowCase* app project, open *StringLibraryTest/UnitTest1.cs*, and select **Debug Tests in current file** between lines 7 and 8. If you're unable to find it, press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd> to open the command palette and enter **Reload Window**.

Visual Studio Code starts the test project with the debugger attached. Execution will stop at any breakpoint you've added to the test project or the underlying library code.

## Additional resources

- [Unit testing in .NET](../testing/index.md)

## Next steps

In this tutorial, you unit tested a class library. You can make the library available to others by publishing it to [NuGet](https://nuget.org) as a package. To learn how, follow a NuGet tutorial:

> [!div class="nextstepaction"]
> [Create and publish a package using the dotnet CLI](/nuget/quickstart/create-and-publish-a-package-using-the-dotnet-cli)

If you publish a library as a NuGet package, others can install and use it. To learn how, follow a NuGet tutorial:

> [!div class="nextstepaction"]
> [Install and use a package using the dotnet CLI](/nuget/quickstart/install-and-use-a-package-using-the-dotnet-cli)

A library doesn't have to be distributed as a package. It can be bundled with a console app that uses it. To learn how to publish a console app, see the earlier tutorial in this series:

> [!div class="nextstepaction"]
> [Publish a .NET console application using Visual Studio Code](publishing-with-visual-studio-code.md)

The Visual Studio Code extension C# Dev Kit provides more tools for developing C# apps and libraries:

> [!div class="nextstepaction"]
> [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp-dev-kit)
