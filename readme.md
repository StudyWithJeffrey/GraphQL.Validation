<!--
GENERATED FILE - DO NOT EDIT
This file was generated by [MarkdownSnippets](https://github.com/SimonCropp/MarkdownSnippets).
Source File: /readme.source.md
To change this file edit the source file and then run MarkdownSnippets.
-->

# <img src="/src/icon.png" height="30px"> GraphQL.Validation

[![Build status](https://ci.appveyor.com/api/projects/status/wvk8wm3n227b2b3q/branch/master?svg=true)](https://ci.appveyor.com/project/SimonCropp/graphql-validation)
[![NuGet Status](https://img.shields.io/nuget/v/GraphQL.FluentValidation.svg)](https://www.nuget.org/packages/GraphQL.FluentValidation/)

Add [FluentValidation](https://fluentvalidation.net/) support to [GraphQL.net](https://github.com/graphql-dotnet/graphql-dotnet)

Support is available via a [Tidelift Subscription](https://tidelift.com/subscription/pkg/nuget-graphql.fluentvalidation?utm_source=nuget-graphql.fluentvalidation&utm_medium=referral&utm_campaign=enterprise).

<!-- toc -->
## Contents

  * [Usage](#usage)
    * [Define validators](#define-validators)
    * [Setup Validators](#setup-validators)
    * [Add to ExecutionOptions](#add-to-executionoptions)
    * [UserContext must be a dictionary](#usercontext-must-be-a-dictionary)
    * [Trigger validation](#trigger-validation)
    * [Difference from IValidationRule](#difference-from-ivalidationrule)
  * [Testing](#testing)
    * [Integration](#integration)
    * [Unit](#unit)
  * [Security contact information](#security-contact-information)<!-- endtoc -->


## NuGet package

https://nuget.org/packages/GraphQL.FluentValidation/


## Usage


### Define validators

Given the following input:

<!-- snippet: input -->
<a id='snippet-input'/></a>
```cs
public class MyInput
{
    public string Content { get; set; } = null!;
}
```
<sup><a href='/src/SampleWeb/Graphs/MyInput.cs#L1-L8' title='File snippet `input` was extracted from'>snippet source</a> | <a href='#snippet-input' title='Navigate to start of snippet `input`'>anchor</a></sup>
<!-- endsnippet -->

And graph:

<!-- snippet: graph -->
<a id='snippet-graph'/></a>
```cs
public class MyInputGraph :
    InputObjectGraphType
{
    public MyInputGraph()
    {
        Field<StringGraphType>("content");
    }
}
```
<sup><a href='/src/SampleWeb/Graphs/MyInputGraph.cs#L3-L12' title='File snippet `graph` was extracted from'>snippet source</a> | <a href='#snippet-graph' title='Navigate to start of snippet `graph`'>anchor</a></sup>
<!-- endsnippet -->

A custom validator can be defined as follows:

<!-- snippet: validator -->
<a id='snippet-validator'/></a>
```cs
public class MyInputValidator :
    AbstractValidator<MyInput>
{
    public MyInputValidator()
    {
        RuleFor(_ => _.Content)
            .NotEmpty();
    }
}
```
<sup><a href='/src/SampleWeb/Graphs/MyInputValidator.cs#L3-L13' title='File snippet `validator` was extracted from'>snippet source</a> | <a href='#snippet-validator' title='Navigate to start of snippet `validator`'>anchor</a></sup>
<!-- endsnippet -->


### Setup Validators

Validators need to be added to the `ValidatorTypeCache`. This should be done once at application startup.

<!-- snippet: StartConfig -->
<a id='snippet-startconfig'/></a>
```cs
var validatorTypeCache = new ValidatorTypeCache();
validatorTypeCache.AddValidatorsFromAssembly(assemblyContainingValidators);
var schema = new Schema();
var executer = new DocumentExecuter();
```
<sup><a href='/src/Tests/Snippets/QueryExecution.cs#L18-L25' title='File snippet `startconfig` was extracted from'>snippet source</a> | <a href='#snippet-startconfig' title='Navigate to start of snippet `startconfig`'>anchor</a></sup>
<!-- endsnippet -->

Generally `ValidatorTypeCache` is scoped per app and can be collocated with `Schema`, `DocumentExecuter` initialization.

Dependency Injection can be used for validators. Create a `ValidatorTypeCache` with the
`useDependencyInjection: true` parameter and call one of the `AddValidatorsFrom*` methods from
[FluentValidation.DependencyInjectionExtensions](https://www.nuget.org/packages/FluentValidation.DependencyInjectionExtensions/)
package in the `Startup`. By default, validators are added to the DI container with a transient lifetime.


### Add to ExecutionOptions

Validation needs to be added to any instance of `ExecutionOptions`.

<!-- snippet: UseFluentValidation -->
<a id='snippet-usefluentvalidation'/></a>
```cs
var options = new ExecutionOptions
{
    Schema = schema,
    Query = queryString,
    Inputs = inputs
}
.UseFluentValidation(validatorTypeCache);

var executionResult = await executer.ExecuteAsync(options);
```
<sup><a href='/src/Tests/Snippets/QueryExecution.cs#L30-L42' title='File snippet `usefluentvalidation` was extracted from'>snippet source</a> | <a href='#snippet-usefluentvalidation' title='Navigate to start of snippet `usefluentvalidation`'>anchor</a></sup>
<!-- endsnippet -->


### UserContext must be a dictionary

This library needs to be able to pass the list of validators, in the form of `ValidatorTypeCache`, through the graphql context. The only way of achieving this is to use the `ExecutionOptions.UserContext`. To facilitate this, the type passed to `ExecutionOptions.UserContext` has to implement `IDictionary<string, object>`. There are two approaches to achieving this:


#### 1. Have the user context class implement IDictionary

Given a user context class of the following form:

<!-- snippet: ContextImplementingDictionary -->
<a id='snippet-contextimplementingdictionary'/></a>
```cs
public class MyUserContext :
    Dictionary<string, object>
{
    public MyUserContext(string myProperty)
    {
        MyProperty = myProperty;
    }

    public string MyProperty { get; }
}
```
<sup><a href='/src/Tests/Snippets/QueryExecution.cs#L45-L58' title='File snippet `contextimplementingdictionary` was extracted from'>snippet source</a> | <a href='#snippet-contextimplementingdictionary' title='Navigate to start of snippet `contextimplementingdictionary`'>anchor</a></sup>
<!-- endsnippet -->

The `ExecutionOptions.UserContext` can then be set as follows:

<!-- snippet: ExecuteQueryWithContextImplementingDictionary -->
<a id='snippet-executequerywithcontextimplementingdictionary'/></a>
```cs
var options = new ExecutionOptions
{
    Schema = schema,
    Query = queryString,
    Inputs = inputs,
    UserContext = new MyUserContext
    (
        myProperty: "the value"
    )
}
.UseFluentValidation(validatorTypeCache);
```
<sup><a href='/src/Tests/Snippets/QueryExecution.cs#L62-L76' title='File snippet `executequerywithcontextimplementingdictionary` was extracted from'>snippet source</a> | <a href='#snippet-executequerywithcontextimplementingdictionary' title='Navigate to start of snippet `executequerywithcontextimplementingdictionary`'>anchor</a></sup>
<!-- endsnippet -->


#### 2. Have the user context class exist inside a IDictionary

<!-- snippet: ExecuteQueryWithContextInsideDictionary -->
<a id='snippet-executequerywithcontextinsidedictionary'/></a>
```cs
var options = new ExecutionOptions
{
    Schema = schema,
    Query = queryString,
    Inputs = inputs,
    UserContext = new Dictionary<string, object>
    {
        {
            "MyUserContext",
            new MyUserContext
            (
                myProperty: "the value"
            )
        }
    }
}
.UseFluentValidation(validatorTypeCache);
```
<sup><a href='/src/Tests/Snippets/QueryExecution.cs#L81-L101' title='File snippet `executequerywithcontextinsidedictionary` was extracted from'>snippet source</a> | <a href='#snippet-executequerywithcontextinsidedictionary' title='Navigate to start of snippet `executequerywithcontextinsidedictionary`'>anchor</a></sup>
<!-- endsnippet -->


#### No UserContext

If no instance is passed to `ExecutionOptions.UserContext`:

<!-- snippet: NoContext -->
<a id='snippet-nocontext'/></a>
```cs
var options = new ExecutionOptions
{
    Schema = schema,
    Query = queryString,
    Inputs = inputs
}
.UseFluentValidation(validatorTypeCache);
```
<sup><a href='/src/Tests/Snippets/QueryExecution.cs#L106-L116' title='File snippet `nocontext` was extracted from'>snippet source</a> | <a href='#snippet-nocontext' title='Navigate to start of snippet `nocontext`'>anchor</a></sup>
<!-- endsnippet -->

Then the `UseFluentValidation` method will instantiate it to a new `Dictionary<string, object>`.


### Trigger validation

To trigger the validation, when reading arguments use `GetValidatedArgument` instead of `GetArgument`:

<!-- snippet: GetValidatedArgument -->
<a id='snippet-getvalidatedargument'/></a>
```cs
public class Query :
    ObjectGraphType
{
    public Query()
    {
        Field<ResultGraph>(
            "inputQuery",
            arguments: new QueryArguments(
                new QueryArgument<MyInputGraph>
                {
                    Name = "input"
                }
            ),
            resolve: context =>
            {
                var input = context.GetValidatedArgument<MyInput>("input");
                return new Result
                {
                    Data = input.Content
                };
            }
        );
    }
}
```
<sup><a href='/src/SampleWeb/Query.cs#L4-L31' title='File snippet `getvalidatedargument` was extracted from'>snippet source</a> | <a href='#snippet-getvalidatedargument' title='Navigate to start of snippet `getvalidatedargument`'>anchor</a></sup>
<!-- endsnippet -->


### Difference from IValidationRule

The validation implemented in this project has nothing to do with the validation of the incoming GraphQL
request, which is described in the [official specification](http://spec.graphql.org/June2018/#sec-Validation).
[GraphQL.NET](https://github.com/graphql-dotnet/graphql-dotnet) has a concept of [validation rules](https://github.com/graphql-dotnet/graphql-dotnet/blob/master/src/GraphQL/Validation/IValidationRule.cs)
that would work **before** request execution stage. In this project validation occurs for input arguments
**at the request execution stage**. This additional validation complements but does not replace the standard
set of validation rules.


## Testing

### Integration

A full end-to-en test can be run against the GraphQL controller:

<!-- snippet: GraphQlControllerTests -->
<a id='snippet-graphqlcontrollertests'/></a>
```cs
[UsesVerify]
public class GraphQLControllerTests
{
    [Fact]
    public async Task RunQuery()
    {
        using var server = GetTestServer();
        using var client = server.CreateClient();
        var query = @"
{
  inputQuery(input: {content: ""TheContent""}) {
    data
  }
}
";
        var body = new
        {
            query
        };
        var serialized = JsonConvert.SerializeObject(body);
        using var content = new StringContent(
            serialized,
            Encoding.UTF8,
            "application/json");
        using var request = new HttpRequestMessage(HttpMethod.Post, "graphql")
        {
            Content = content
        };
        using var response = await client.SendAsync(request);
        response.EnsureSuccessStatusCode();
        await Verifier.Verify(await response.Content.ReadAsStringAsync());
    }

    static TestServer GetTestServer()
    {
        var hostBuilder = new WebHostBuilder();
        hostBuilder.UseStartup<Startup>();
        return new TestServer(hostBuilder);
    }
}
```
<sup><a href='/src/SampleWeb.Tests/GraphQlControllerTests.cs#L10-L52' title='File snippet `graphqlcontrollertests` was extracted from'>snippet source</a> | <a href='#snippet-graphqlcontrollertests' title='Navigate to start of snippet `graphqlcontrollertests`'>anchor</a></sup>
<!-- endsnippet -->


### Unit

Unit tests can be run a specific field of a query:

<!-- snippet: QueryTests -->
<a id='snippet-querytests'/></a>
```cs
[UsesVerify]
public class QueryTests
{
    [Fact]
    public Task RunInputQuery()
    {
        var field = new Query().GetField("inputQuery");

        var userContext = new GraphQLUserContext();
        userContext.AddValidatorCache(ValidatorCacheBuilder.Instance);

        var input = new MyInput
        {
            Content = "TheContent"
        };
        var dictionary = input.AsDictionary();
        var fieldContext = new ResolveFieldContext
        {
            Arguments = new Dictionary<string, object>
            {
                {
                    "input", dictionary
                }
            },
            UserContext = userContext
        };
        var result = (Result) field.Resolver.Resolve(fieldContext);
        return Verifier.Verify(result);
    }

    [Fact]
    public Task RunInvalidInputQuery()
    {
        Thread.CurrentThread.CurrentUICulture = new CultureInfo("en-US");
        var field = new Query().GetField("inputQuery");

        var userContext = new GraphQLUserContext();
        userContext.AddValidatorCache(ValidatorCacheBuilder.Instance);
        var fieldContext = new ResolveFieldContext
        {
            Arguments = new Dictionary<string, object>
            {
                {
                    "input", new Dictionary<string, object>()
                }
            },
            UserContext = userContext
        };
        var exception = Assert.Throws<ValidationException>(
            () => field.Resolver.Resolve(fieldContext));
        return Verifier.Verify(exception.Message);
    }

 
}
```
<sup><a href='/src/SampleWeb.Tests/QueryTests.cs#L10-L67' title='File snippet `querytests` was extracted from'>snippet source</a> | <a href='#snippet-querytests' title='Navigate to start of snippet `querytests`'>anchor</a></sup>
<!-- endsnippet -->


## Security contact information

To report a security vulnerability, use the [Tidelift security contact](https://tidelift.com/security). Tidelift will coordinate the fix and disclosure.


## Icon

[Shield](https://thenounproject.com/term/shield/1893182/) designed by [Maxim Kulikov](https://thenounproject.com/maxim221/) from [The Noun Project](https://thenounproject.com)
