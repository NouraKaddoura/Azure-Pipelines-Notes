## Azure-Pipelines-Notes
[Link](https://docs.microsoft.com/en-us/learn/modules/run-quality-tests-build-pipeline/1-introduction)

This module demonstrates how to set up automated testing to help ensure that your latest feature works and that you haven't broken anything along the way.

### The need for automated testing

When we think about automated testing, it's common to separate tests into layers. Mike Cohn proposes this concept, known as the test pyramid, in his book Succeeding with Agile.

When you relate testing to continuous integration and continuous delivery pipelines, two concepts you'll hear about are continuous testing and shifting left.

Continuous testing means tests are run early in the development process and as every change moves through the pipeline. Shifting left means considering software quality and testing earlier in the development process.

### Unit test that tests everything - run fast and run during your build. Add all your units tests. 

Automated tests can serve the same purpose. Automated test code often uses a human-readable format. The set of inputs you provide represent values your users might enter. Each associated output specifies the result your users should expect.

In fact, many developers follow the test-driven development, or TDD, method by writing their test code before implementing a new feature. The idea is to write a set of tests, often called specs, that initially fail.

lint testing, a form of static code analysis, checks your source code to determine whether it conforms to your team's style guide. Code that's formatted consistently is usually easier for everyone to read and maintain.

### unit testing and code coverage testing.

Unit testing verifies the most fundamental components of your program or library, such as an individual function or method. You specify one or more inputs along with the expected results. The test runner performs each test and checks to see whether the actual and expected results match.

### What makes a good test?

- Don't test for the sake of testing: Your tests should serve a purpose beyond being a checklist item to cross off. Write tests that verify that your critical code works as intended and doesn't break existing functionality.
- Keep your tests short: Tests should finish as quickly as possible, especially those that happen during the development and build phases. When tests are run as each change moves through the pipeline, you don't want them to be the bottleneck.
- Ensure that your tests are repeatable: Test runs should produce the same results each time, whether you run them on your computer, a coworker's computer, or in the build pipeline.
- Keep your tests focused: A common misconception is that tests are meant to cover code written by others. Ordinarily, your tests should cover only your code. For example, if you're using an open-source graphics library in your project, you don't need to test that library.
- Choose the right granularity: For example, if you're performing unit testing, an individual test shouldn't combine or test multiple functions or methods. Test each function separately and later write integration tests that verify that multiple components interact properly.

#### For example, you can use Selenium to perform UI testing on many types of web browsers and operating systems.

#### NUnit for unit testing because it's popular in the .NET community
## Add unit tests to your application

#### Example of Unit Test

C#

```

[TestCase("Milky Way")]
[TestCase("Andromeda")]
[TestCase("Pinwheel")]
[TestCase("NGC 1300")]
[TestCase("Messier 82")]
public void FetchOnlyRequestedGameRegion(string gameRegion)
{
    const int PAGE = 0; // take the first page of results
    const int MAX_RESULTS = 10; // sample up to 10 results

    // Form the query predicate.
    // This expression selects all scores for the provided game region.
    Expression<Func<Score, bool>> queryPredicate = score => (score.GameRegion == gameRegion);

    // Fetch the scores.
    Task<IEnumerable<Score>> scoresTask = _scoreRepository.GetItemsAsync(
        queryPredicate, // the predicate defined above
        score => 1, // we don't care about the order
        PAGE,
        MAX_RESULTS
    );
    IEnumerable<Score> scores = scoresTask.Result;

    // Verify that each score's game region matches the provided game region.
    Assert.That(scores, Is.All.Matches<Score>(score => score.GameRegion == gameRegion));
}

```

In an NUnit test method, TestCase provides inline data to use to test that method. Here, NUnit calls the FetchOnlyRequestedGameRegion unit test method like this:

```

FetchOnlyRequestedGameRegion("Milky Way");
FetchOnlyRequestedGameRegion("Andromeda");
FetchOnlyRequestedGameRegion("Pinwheel");
FetchOnlyRequestedGameRegion("NGC 1300");
FetchOnlyRequestedGameRegion("Messier 82");

```

Note `Assert.That` method at the end of the test. An assertion is a condition or statement that you declare to be true.  If the condition turns out to be false, that could indicate a bug in your code. NUnit runs each test method using the inline data you specify and records the result as a passing or failing test.

#### Detached Head issues

```
git fetch origin
git reset --hard origin/master
git clean -f
git checkout master
git pull origin

```

More troubleshooting: https://stackoverflow.com/questions/10228760/fix-a-git-detached-head

#### Switch to unit test branch and pull it from upstream


```
git fetch upstream unit-tests
git checkout -b unit-tests upstream/unit-tests

```

Run the test locally

```
dotnet build --configuration Release
dotnet test --configuration Release --no-build
//with logger
dotnet test Tailspin.SpaceGame.Web.Tests --configuration Release --no-build --logger trx


```

Example of a test

```
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq.Expressions;
using System.Threading.Tasks;
using NUnit.Framework;
using TailSpin.SpaceGame.Web;
using TailSpin.SpaceGame.Web.Models;

namespace Tests
{
    public class DocumentDBRepository_GetItemsAsyncShould
    {
        private IDocumentDBRepository<Score> _scoreRepository;

        [SetUp]
        public void Setup()
        {
            using (Stream scoresData = typeof(IDocumentDBRepository<Score>)
                .Assembly
                .GetManifestResourceStream("Tailspin.SpaceGame.Web.SampleData.scores.json"))
            {
                _scoreRepository = new LocalDocumentDBRepository<Score>(scoresData);
            }
        }

        [TestCase("Milky Way")]
        [TestCase("Andromeda")]
        [TestCase("Pinwheel")]
        [TestCase("NGC 1300")]
        [TestCase("Messier 82")]
        public void FetchOnlyRequestedGameRegion(string gameRegion)
        {
            const int PAGE = 0; // take the first page of results
            const int MAX_RESULTS = 10; // sample up to 10 results

            // Form the query predicate.
            // This expression selects all scores for the provided game region.
            Expression<Func<Score, bool>> queryPredicate = score => (score.GameRegion == gameRegion);

            // Fetch the scores.
            Task<IEnumerable<Score>> scoresTask = _scoreRepository.GetItemsAsync(
                queryPredicate, // the predicate defined above
                score => 1, // we don't care about the order
                PAGE,
                MAX_RESULTS
            );
            IEnumerable<Score> scores = scoresTask.Result;

            // Verify that each score's game region matches the provided game region.
            Assert.That(scores, Is.All.Matches<Score>(score => score.GameRegion == gameRegion));
        }
    }
}

```

After you run on local machine and verify the test works. Add this to the pipeline YML file

```

trigger:
- '*'

pool:
  vmImage: 'ubuntu-16.04'
  demands:
    - npm

variables:
  buildConfiguration: 'Release'
  wwwrootDir: 'Tailspin.SpaceGame.Web/wwwroot'
  dotnetSdkVersion: '3.1.100'

steps:
- task: UseDotNet@2
  displayName: 'Use .NET Core SDK $(dotnetSdkVersion)'
  inputs:
    version: '$(dotnetSdkVersion)'

- task: Npm@1
  displayName: 'Run npm install'
  inputs:
    verbose: false

- script: './node_modules/.bin/node-sass $(wwwrootDir) --output $(wwwrootDir)'
  displayName: 'Compile Sass assets'

- task: gulp@1
  displayName: 'Run gulp tasks'

- script: 'echo "$(Build.DefinitionName), $(Build.BuildId), $(Build.BuildNumber)" > buildinfo.txt'
  displayName: 'Write build info'
  workingDirectory: $(wwwrootDir)

- task: DotNetCoreCLI@2
  displayName: 'Restore project dependencies'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build the project - $(buildConfiguration)'
  inputs:
    command: 'build'
    arguments: '--no-restore --configuration $(buildConfiguration)'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Run unit tests - $(buildConfiguration)'
  inputs:
    command: 'test'
    arguments: '--no-build --configuration $(buildConfiguration)'
    publishTestResults: true
    projects: '**/*.Tests.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Publish the project - $(buildConfiguration)'
  inputs:
    command: 'publish'
    projects: '**/*.csproj'
    publishWebProjects: false
    arguments: '--no-build --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/$(buildConfiguration)'
    zipAfterPublish: true

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
  condition: succeeded()
  
```

This is the piece to add:

```
- task: DotNetCoreCLI@2
  displayName: 'Run unit tests - $(buildConfiguration)'
  inputs:
    command: 'test'
    arguments: '--no-build --configuration $(buildConfiguration)'
    publishTestResults: true
    projects: '**/*.Tests.csproj'
    
```

**Notice that this task does not specify the --logger trx argument that you used when you ran the tests manually. The publishTestResults argument adds that for you. This argument tells the pipeline to generate the TRX file to a temporary directory, accessible through the $(Agent.TempDirectory) built-in variable. It also publishes the task results to the pipeline.**

Now upload to github to the unit-test branch

```
git add azure-pipelines.yml
git commit -m "Run and publish unit tests"
git push origin unit-tests

```
![link](https://docs.microsoft.com/en-us/learn/azure-devops/run-quality-tests-build-pipeline/media/4-pipeline-task.png)

When you target .NET Core applications to run on Linux, coverlet  is a popular option. Coverlet is a cross-platform, code-coverage library for .NET Core. Before we add code coverage to the pipeline, let's check in with the team.

[Coverlet](https://github.com/coverlet-coverage/coverlet)

Code coverage. That will tell us the percentage of our code that has unit tests. We can use a tool called "coverlet" to collect coverage information when the tests run.

https://www.nuget.org/packages/coverlet.msbuild


verify the process manually first

```
dotnet tool install --global dotnet-reportgenerator-globaltool --version 4.1.1

dotnet test --no-build \
  --configuration Release \
  /p:CollectCoverage=true \
  /p:CoverletOutputFormat=cobertura \
  /p:CoverletOutput=./TestResults/Coverage/
  
```

#### If the command fails run this:

```
MSYS2_ARG_CONV_EXCL="*" dotnet test --no-build \
  --configuration Release \
  /p:CollectCoverage=true \
  /p:CoverletOutputFormat=cobertura \
  /p:CoverletOutput=./TestResults/Coverage/
  
```

Run the following `reportgenerator` command to convert the Cobertura file to HTML:

```
$HOME/.dotnet/tools/reportgenerator \
  -reports:./Tailspin.SpaceGame.Web.Tests/TestResults/Coverage/coverage.cobertura.xml \
  -targetdir:./CodeCoverage \
  -reporttypes:HtmlInline_AzurePipelines
  
```

Copy the file path to view it in the browser: /Users/nourakaddoura/Desktop/DevOpsTraining/microsoftTraining/mslearn-tailspin-spacegame-web/CodeCoverage/index.htm

- Now add the steps the yml file but first create a new branch
Terminal:
`git checkout -b code-coverage`

YML:

```
trigger:
- '*'

pool:
  vmImage: 'ubuntu-16.04'
  demands:
    - npm

variables:
  buildConfiguration: 'Release'
  wwwrootDir: 'Tailspin.SpaceGame.Web/wwwroot'
  dotnetSdkVersion: '3.1.100'

steps:
- task: UseDotNet@2
  displayName: 'Use .NET Core SDK $(dotnetSdkVersion)'
  inputs:
    version: '$(dotnetSdkVersion)'

- task: Npm@1
  displayName: 'Run npm install'
  inputs:
    verbose: false

- script: './node_modules/.bin/node-sass $(wwwrootDir) --output $(wwwrootDir)'
  displayName: 'Compile Sass assets'

- task: gulp@1
  displayName: 'Run gulp tasks'

- script: 'echo "$(Build.DefinitionName), $(Build.BuildId), $(Build.BuildNumber)" > buildinfo.txt'
  displayName: 'Write build info'
  workingDirectory: $(wwwrootDir)

- task: DotNetCoreCLI@2
  displayName: 'Restore project dependencies'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build the project - $(buildConfiguration)'
  inputs:
    command: 'build'
    arguments: '--no-restore --configuration $(buildConfiguration)'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Install ReportGenerator'
  inputs:
    command: custom
    custom: tool
    arguments: 'install --global dotnet-reportgenerator-globaltool'

- task: DotNetCoreCLI@2
  displayName: 'Run unit tests - $(buildConfiguration)'
  inputs:
    command: 'test'
    arguments: '--no-build --configuration $(buildConfiguration) /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura /p:CoverletOutput=$(Build.SourcesDirectory)/TestResults/Coverage/'
    publishTestResults: true
    projects: '**/*.Tests.csproj'

- script: |
    reportgenerator -reports:$(Build.SourcesDirectory)/**/coverage.cobertura.xml -targetdir:$(Build.SourcesDirectory)/CodeCoverage -reporttypes:HtmlInline_AzurePipelines
  displayName: 'Create code coverage report'

- task: PublishCodeCoverageResults@1
  displayName: 'Publish code coverage report'
  inputs:
    codeCoverageTool: 'cobertura'
    summaryFileLocation: '$(Build.SourcesDirectory)/**/coverage.cobertura.xml'

- task: DotNetCoreCLI@2
  displayName: 'Publish the project - $(buildConfiguration)'
  inputs:
    command: 'publish'
    projects: '**/*.csproj'
    publishWebProjects: false
    arguments: '--no-build --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/$(buildConfiguration)'
    zipAfterPublish: true

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: drop'
  condition: succeeded()
  
```

push up to github

```
git add azure-pipelines.yml
git commit -m "Add code coverage"
git push origin code-coverage

```

If you want to add code coverage to the dashboard: https://docs.microsoft.com/en-us/learn/modules/run-quality-tests-build-pipeline/6-perform-code-coverage

remove code coverage folder from root directory: `rm -rf CodeCoverage/`

get the failed branch

#### Fix a failed test

```
git fetch upstream failed-test
git checkout -b failed-test upstream/failed-test
git commit --allow-empty -m "Trigger Azure Pipelines"
git push origin failed-test

```

see the test failure on azure pipelines.

reproduce the failure locally:

```
dotnet build --configuration Release
dotnet test --no-build --configuration Release


```

Examine changes and fix the error. in this case Tailspin.SpaceGame.Web/LocalDocumentDBRepository.cs pagesize

Defining build tasks locally first helps you understand and verify the process before you add build tasks to your pipeline.

Remember, the process you followed was specific to .NET Core applications. The tools and tasks that you use depend on the programming language and frameworks that you use to build your applications.

Unit Tests: https://docs.microsoft.com/en-us/learn/modules/run-quality-tests-build-pipeline/9-summary
