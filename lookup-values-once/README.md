## Background

In this use case, the professor wants a way that students can submit a set of values once and have a custom calculation 
performed on them only once.

This replaces a homework exercise where teams of students would submit a set of values to a human (usually the professor),
who would manually perform calculations/lookups using these values with a custom Excel spreadsheet, and send them back.

This setup has a number of caveats:

1. It's a manually intensive process for the professor.
2. If the professor wants to be responsive to their students, they need to be available and working during the period that students can submit their answers.
3. Because of the number of students in the class and the arduous nature of the data processing, the professor usually limits this exercise to groups of students where each group submits one collected set of answers. 

## Motivation

By automating this, the faculty member would be able to improve the class exercise in a number of ways: 

1. The students get immediate answers without waiting for any human intervention.
2. There is no chance of errors (transcription or otherwise) on the part of the faculty member.
3. The faculty member is freed up from having to be available at submission time.
4. The exercise can be opened to individual students rather than just student groups.

## Result

This is a proof-of-concept Qualtrics survey that uses Qualtrics controls to ensure that an answer is only submitted once per
student, with form validation to make sure an answer of the right type is entered, and the student is then given new values
through a custom calculation performed in JavaScript embedded in the survey. 

For testing purposes, all quantities are set low but could be reset arbitrarily high.

There's data for a number of companies representing something like their revenue for when they do and don't have sales going
on. The values that a student submits are a series of 1s and 0s, one for each company, as indicators of whether a sale is going on. 

In this example, there are only 5 companies, and form validation on the first question ensures that the student only
submitted 5 values. There must be the same number of input values as there are companies. The 0 or 1 inputs must be
separated by carriage returns/new lines in the input text area. There must also be a carriage return after the last line.

### Live Survey Example

[https://kellogg.qualtrics.com/jfe/form/SV_6G1HRVZOU1hiCDX](https://kellogg.qualtrics.com/jfe/form/SV_6G1HRVZOU1hiCDX)

## Qualtrics Survey Construction

Two questions, each are a multiline text entry, separated by a page break.

### Survey Flow

Add Embedded Data called `sale_choices`.

### Survey Options

1. Survey Experience
  1. `Back Button` - Disable
  2. `Save and Continue` - Disable
2. Survey Protection
  1. Select `By Invitation Only`
  2. Select `Prevent Ballot Box Stuffing`

### Question 1

* Question Name: QSalesIndicators
* Question ID: QID1
* Question Type: Text Entry
* Text Type: Multi Line
* Validation Options: Force Response
* Validation Type: Custom Validation
* Custom Validation - rule 1: question choice Matches Regex `^([0-1]\n){4}[0-1][\n]*$` - exactly 5 lines with either a 0 or 1 on each
* Custom Validation - (AND) rule 2: question choice Matches Regex `^(1[0|\n]*){3}[0|\n]*$` - exactly 3 of the lines are a 1
* Custom Validation - error message: [Custom Message](https://www.qualtrics.com/support/survey-platform/survey-module/editing-questions/validation/#CustomValidationMessages)

#### Embedded JavaScript

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
});

Qualtrics.SurveyEngine.addOnReady(function()
{
});

Qualtrics.SurveyEngine.addOnUnload(function()
{
    var v = document.getElementById("QR~QID1").value;
    var new_e = v.strip().replace(/\n/g, ' ');
    Qualtrics.SurveyEngine.setEmbeddedData('sale_choices', new_e);
    console.log('Page unloaded!');
    console.log('embedded data sale_choices set to ' + new_e);
});
```

### Question 2

* Question Name: QSalesData
* Question ID: QID2
* Question Type: Text Entry
* Text Type: Multi Line

#### Embedded JavaScript

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
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
    
    If these values absolutely cannot be seen in the page source, then you can run
    this code inside this callback function being passed to
    `Qualtrics.SurveyEngine.addOnload()` through a JavaScript Obfuscator like [this
    one](https://www.danstools.com/javascript-obfuscate/index.php).
    
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
});
```

#### Obfuscated Embedded JavaScript

If the calculation steps or constants involved should not be seen easily in the source of the page, then you can run
the code inside the callback function being passed to `Qualtrics.SurveyEngine.addOnload()` through a JavaScript Obfuscator
like [this one](https://www.danstools.com/javascript-obfuscate/index.php). You would get something like this.

```javascript
Qualtrics.SurveyEngine.addOnload(function()
{
    eval(function(p,a,c,k,e,d){e=function(c){return(c<a?'':e(parseInt(c/a)))+((c=c%a)>35?String.fromCharCode(c+29):c.toString(36))};if(!''.replace(/^/,String)){while(c--){d[e(c)]=k[c]||e(c)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('3 5=[[t,o],[j,q],[m,l],[k,p],[s,h]];3 2=b.c.g("2");9.a("d e f r!");9.a("w C 2 = "+2);2=2.u(" ");3 4=D();E(3 i=0;i<2.F;i++){7(2[i]=="1"){4.6(5[i][0])}B 7(2[i]=="0"){4.6(5[i][1])}}3 8=4.x("\\n");z.G("y~v").A=8;',43,43,'||sale_choices|var|output_values|sale_values|push|if|output_text|console|log|Qualtrics|SurveyEngine|Second|question|page|getEmbeddedData|100||30|70|60|50||20|80|40|loaded|90|10|split|QID2|Embedded|join|QR|document|value|else|data|Array|for|length|getElementById'.split('|'),0,{}))
});

Qualtrics.SurveyEngine.addOnReady(function()
{
});

Qualtrics.SurveyEngine.addOnUnload(function()
{
});
```
