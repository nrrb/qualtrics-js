## Background

In this use case, the professor wants a way that students can submit a set of values once and have a custom calculation 
performed on them only once. This replaces a homework exercise where teams of students would submit a set of values to a
human (usually the professor), who would manually perform calculations/lookups using these values with a custom Excel
spreadsheet, and send them back. This was a manually intensive process, required the professor to be available to process
answers in real-time at submission time, and limited submissions to being on behalf of student groups instead of
individuals to limit the work the professor would have to do.

## Motivation

By putting this calculation into JavaScript embedded in Qualtrics, utilizing Qualtrics controls to prevent students from
re-submitting, the faculty member is able to improve the class exercise in a number of ways: the students get immediate
answers without waiting for any human intervention, there is no chance of errors (transcription or otherwise) on the part
of the faculty member, the faculty member is freed up from having to be available at submission time, and the exercise
can be opened to individual students rather than just student groups.

## Qualtrics Survey Construction

Two questions, each are a multiline text entry, separated by a page break.

### Question 1

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
});

Qualtrics.SurveyEngine.addOnReady(function()
{
    /*Place your JavaScript here to run when the page is fully displayed*/

});

Qualtrics.SurveyEngine.addOnUnload(function()
{
    /*Place your JavaScript here to run when the page is unloaded*/
    var v = document.getElementById("QR~QID1").value;
    var new_e = v.strip().replace(/\n/g, ' ');
    Qualtrics.SurveyEngine.setEmbeddedData('sale_choices', new_e);
    console.log('Page unloaded!');
    console.log('embedded data sale_choices set to ' + new_e);
});
```

### Question 2

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
    /*Place your JavaScript here to run when the page loads*/
    /*
    Rows in the sale_values array correspond to companies. The value in the first
    column corresponds to when there is a sale occurring at the given company. The
    value in the second column corresponds to when there is not a sale.

    The values that the user inputs are a 1 or 0 for each company, with 1 indicating
    that there is a sale occurring and 0 indicating that there is not.

    In this example, there are only 5 companies, and form validation on the first
    question ensures that the student only submitted 5 values. There must be the
    same number of input values as there are companies. The 0 or 1 inputs must be
    separated by carriage returns/new lines in the input text area.
    */
    var sale_values = [
        [10,20],
        [30,40],
        [50,60],
        [70,80],
        [90,100]
        ];

    var sale_choices = Qualtrics.SurveyEngine.getEmbeddedData("sale_choices");
    /*
    Some diagnostic messages to let us know where we (dev) are and that we're getting
    the expected values.
    */
    console.log("Second question page loaded!");
    console.log("Embedded data sale_choices = " + sale_choices);
    /*
    In the JavaScript of the first question page, the newlines in the input were replaced
    by spaces. Now, we separate the values using the space as delimiter, and interpret
    the values to choose from the chosen array of sale values per company.
    */
    sale_choices = sale_choices.split(" ");
    var output_values = Array();
    for ( var i = 0; i < sale_choices.length; i++ ) {
        if ( sale_choices[i] == "1" ) {
            output_values.push(sale_values[i][0]);
        } else if ( sale_choices[i] == "0") {
            output_values.push(sale_values[i][1]);
        }
    }
    /*
    Prepare these values for output to question #2, which the student will select,
    copy, and paste back in to their Excel spreadsheet. Because of this, the values
    should be on separate lines so that Excel automatically puts them into separate
    cells in one column.
    */
    var output_text = output_values.join("\n");
    document.getElementById("QR~QID2").value = output_text;
});

Qualtrics.SurveyEngine.addOnReady(function()
{
});

Qualtrics.SurveyEngine.addOnUnload(function()
{
    /*Place your JavaScript here to run when the page is unloaded*/

});
```
