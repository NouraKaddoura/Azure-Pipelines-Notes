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
