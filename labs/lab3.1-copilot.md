# Lab 2 - Pair Program with GitHub Copilot!

Speed up your developer productivity with [GitHub Copilot](https://github.com/features/copilot).

Code faster with less work by leveraging GitHub Copilot’s ability to convert comments to code, recognize patterns, and make alternative suggestions.

Let's explore how you can pair program with AI in ourb.NET 7 web app. The best part is..you don’t have to be proficient in GitHub Copilot!

## Exercise 1 - Prompting GitHub Copilot

GitHub Copilot is an extension that can be installed in Visual Studio, Visual Studio Code, Neovim, and JetBrains IDEs. 

Fun fact: GitHub Copilot can provide suggestions for numerous languages and a wide variety of frameworks, but works especially well for Python, JavaScript, TypeScript, Ruby, Go, C# and C++.

Let’s try to prompt GitHub Copilot to suggest some code for us! 

### Installation 
1. In our last lab while setting up our dev container, we enabled the GitHub Copilot extension to be automagically installed. Let’s double check by searching for GitHub Copilot in our extension list. 
2. Create a new directory at the root of the project called BinarySearchAlgorith.cs. (This won’t be related to our .NET project. We’re just going to use this to get familiar with GitHub Copilot).
3. Write a comment that says `# Write a program that executes a binary search algorithm`. If we press tab, we’ll see that our results may vary. It may not even generate the right algorithm. However, if we get more specific, we can better ensure that GitHub Copilot provides us with the correct results. 
4. Let’s write a more specific, guided comment for GitHub Copilot. This one should do the trick
```
/* Write a function that performs a binary search algorithm in C#
1. Create an array of integers 1, 2, 3, 4, 5
2. The target number should be 4
3. The algorithm should return the index of the target number
*/
```
Here we are providing GitHub Copilot with more examples so it can produce more refined results. 
5. Below write a specific comment prompting it to define the function like this:
```
Define function
```

Now, if we press tab and accept the incoming suggestions from GitHub Copilot, it should generate the desired function.
Your final results should look something like
```csharp
/* Write a function that performs a binary search algorithm in C#
1. Create an array of integers 1, 2, 3, 4, 5
2. The target number should be 4
3. The algorithm should return the index of the target number
*/
 
dotnet tool install -g dotnet-script

dotnet script Demo.cs

/* Write a function that performs a binary search algorithm in C#
1. Create an array of integers 1, 2, 3, 4, 5
2. The target number should be 4
3. The algorithm should return the index of the target number
*/
 
// define function
int binarySearch(int[] arr, int l, int r, int x)
{
    if (r >= l)
    {
        int mid = l + (r - l) / 2;
 
        // If the element is present at the middle itself
        if (arr[mid] == x)
            return mid;
 
        // If element is smaller than mid, then it can only
        // be present in left subarray
        if (arr[mid] > x)
            return binarySearch(arr, l, mid - 1, x);
 
        // Else the element can only be present in right
        // subarray
        return binarySearch(arr, mid + 1, r, x);
    }
 
    // We reach here when element is not present in array
    return -1;
}

// call function
int[] arr = { 1, 2, 3, 4, 5 };
int n = arr.Length;
int x = 4;

int result = binarySearch(arr, 0, n - 1, x);

if (result == -1)
    Console.WriteLine("Element not present");
else
    Console.WriteLine("Element found at index "
                      + result);
```
6. Let’s prompt it to call the function with a comment that says:
```
Call function
``
7. We can run the code to see that it works by running the following commands in our terminal
```
dotnet tool install -g dotnet-script

```
This will give us the ability to run this single file
```
dotnet script BinarySearchAlgorith.cs
```
To actually run the file. 

### What we learned
We just learned how to convert comments to code and how to create better comments that will help us generate our desired results. Now, let’s add code to our actual project

## Exercise 2 - Using the codebase’s existing context

We’re going to add a new column for ratings in our web application 
1. cd into the current folder src/ReadingTime6.Web/Views/Book
2. Inside that directory, open the file /Index.cshtml.
3. In that file on line 27, we’ll write a comment that says, 
```csharp
// add a heading for ratings
```
4. Press tab, and GitHub Copilot will generate the following lines:
```
            <th>
                @Html.DisplayNameFor(model => model.Ratings)

```
5. On line 47, we’ll write a comment that says, 
```csharp
// add a column for ratings
```
6. Press tab, and GitHub Copilot will generate the following lines:
```
         <td>
                    @Html.DisplayFor(modelItem => item.Ratings)
                </td>

```
But it won’t work, just yet! We might even get a few errors because the rating variable isn’t defined in other parts of the project.
7. Let’s define it in our Book model. We can find that in this file `src/ReadingTime6.Web/Models/Book.cs`
8. On line 14 of Book.cs, we’ll write a comment that says
```
Add a property for ratings
```
9. Press `Enter` and then `Tab` to accept GitHub Copilot’s suggestion
```
   public int Ratings { get; set; }
```
10. On line 34 of Book.cs, we’ll write a comment that says, 
```
// add a constructor that takes a title, author, cover, and ratings
```
11. Press `Enter` and then `Tab` to accept GitHub Copilot’s suggestion

```
        public Book(string title, string author, string cover, int ratings)
        {
            Title = title;
            Author = author;
            Cover = cover;
            Ratings = ratings;
        }

```
12. In BookService.cs, we’ll generate the data that it will pull. On line 9, write a comment taht says
```
Add ratings to each book
```
13. Now to each book instance, add a comma, press space and accept GitHub Copilot’s suggestions. It should suggest a random integer for each rating and generate similar results
```
{
            new Book("Accelerate: The Science of Lean Software and DevOps: Building and Scaling High Performing Technology Organizations", "Nicole Forsgren, PhD", "forsgren.jpg", 5),
            new Book("Scrum: The Art of Doing Twice the Work in Half the Time", "Jeff Sutherland", "scrum.jpg", 4),
            new Book("The Lean Startup: How Constant Innovation Creates Radically Successful Businesses","Eric Ries", "lean.jpg", 4),
            new Book("Crossing the Chasm", "Geoffrey A. Moore", "chasm.jpg", 3),
            new Book("The Pragmatic Programmer: From Journeyman to Master", "David Thomas", "pragmatic.jpg", 5),
            new Book("Don't Make Me Think, Revisited: A Common Sense Approach to Web Usability"," Steve Krug", "think.jpg", 4),
        };

```
14. Now, we can rerun the project and see the results in our browser!

### Exercise 3 - Add Unit Tests
1. 
1. In BookTests.cs, add a comment that says, 
``` 
Add unit test for rating property
```
2. Press `Enter` then `Tab` until it generates the following
```csharp
 [Theory]
        [InlineData(1)]
        [InlineData(2)]
        [InlineData(3)]

        public void ExpectValidBookRatingsModelState(int ratings)
        {
            Book book = new Book("Crossing the Chasm", "Geoffrey A.Moore", "cover.jpg", ratings);
            var context = new ValidationContext(book, null, null);
            var result = new List<ValidationResult>();

            var valid = Validator.TryValidateObject(book, context, result, true);

            Assert.True(valid);

        }

```


