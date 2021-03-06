---
layout: post
title: Finding a replacement for the Crystal Reports viewer in an ASP.NET MVC 4 site
comments: False
---

# {{ page.title }}

_28 Jan 2015_

OK... this one was easy.

For starters, the objective was to find an easy to code and maintain a replacement for a WPF MVVM system (running as a [WPF XAML Browser Application](https://msdn.microsoft.com/en-us/library/aa970060(v=vs.90).aspx)) that, among other functionality, allows for an user to open a "window" in the browser, input/choose some filters and preview a report that he or she could export to PDF or print.

Those filters are the usual kind: drop down lists, check boxes and radio buttons, date pickers, numeric spinners and normal text input boxes. Some come pre-filled (like the lists' items that come from the database) and many are checked against the usual constraints: required, maximum length and pattern matching for example.

I first created a view model class that represented the filters. I also used this class to decorate any constrained members with the necessary data annotations. This step was mostly copy-and-paste. I had to replace some data types (like the `ObservableCollection` that was replaced by `ICollection<SelectListItem>`) but nothing major.

Example:

``` cs
public class StatementReportViewModel {
    public ICollection<SelectListItem> Funds { get; set; }
    [Required]
    public decimal UnitPrice { get; set; }
    public DateTime? StartDate { get; set; }
    public DateTime? EndDate { get; set; }
}
```

After that I created a view and coded a simple form:

``` html
@model StatementReportViewModel
@{
    ViewBag.Title = "Statement report"
}

<h2>Statement report</h2>

@using (Html.BeginForm("Show", "StatementReport", FormMethod.Post, new { @class = "form-horizontal", role = "form", target = "_blank" }))
{
    @Html.AntiForgeryToken()
    <p>
        <input type="submit" value="Preview" class="btn btn-default" />
    </p>
    @Html.ValidationSummary("", new { @class = "text-danger" })
    <div class="form-group">
        @Html.LabelFor(m => m.Funds, new { @class = "control-label col-md-2"})
        <div class="col-md-10">
        @Html.DropDownListFor(m => m.Funds, Model.Funds, new { @class = "form-control" })
        </div>
    </div>
    <div class="form-group">
        @Html.LabelFor(m => m.StartDate, new { @class = "col-md-2 control-label" })
        <div class="col-md-2">
            @Html.EditorFor(m => m.StartDate, new { htmlAttributes = new { @class = "form-control", type = "date" } })
            @Html.ValidationMessageFor(m => m.StartDate, "", new { @class = "text-danger" })
        </div>
    </div>
    <div class="form-group">
        @Html.LabelFor(m => m.EndDate, new { @class = "col-md-2 control-label" })
        <div class="col-md-2">
            @Html.EditorFor(m => m.EndDate, new { htmlAttributes = new { @class = "form-control", type = "date" } })
            @Html.ValidationMessageFor(m => m.EndDate, "", new { @class = "text-danger" })
        </div>
    </div>}
    <div class="form-group">
        @Html.LabelFor(m => m.UnitPrice, new { @class = "col-md-2 control-label" })
        <div class="col-md-2">
            @Html.EditorFor(m => m.UnitPrice, new { htmlAttributes = new { @class = "form-control", type = "number", step = "0.01" } })
            @Html.ValidationMessageFor(m => m.UnitPrice, "", new { @class = "text-danger" })
        </div>
    </div>
}
<div>
    @Html.ActionLink("Back", "Index")
</div>
@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}
```

To be able to render the report, I had to add the `CrystalDecisions.CrystalReports.Engine`, `CrystalDecisions.ReportSource` and `CrystalDecisions.Shared` assemblies to the project's references.

Then I copied both the dataset and the report files from the existing WPF project and simply pasted them in the new project (pasted the dataset inside the `Models` folder and the report inside the `Reports` folder that I had previously created).

Note that the report has to have a build action of `Content` (it was embedded in the WPF project). I also use `Copy if newer` so that it is properly updated when I build the project.

I also manually changed the namespaces to match the new project's namespace.

Later, I added a new MVC controller (an empty one) and added two methods: an `Index` one (to show a view with the filters) and a `Show` method (so the page can `POST` the filters and get the report back).

``` cs
public class StatementReportController : Controller {
    public ActionResult Index() {
        var model = new StatementReportViewModel();
        // Fill the model and return a View with it (not showing it here and not
        // using async for now).
        return View(model);
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public ActionResult Show(
        int? funds,
        decimal unitPrice,
        DateTime? startDate,
        DateTime? endDate
    ) {
        var dataset = new StatementReportDataSet();
        // Fill the dataset with the data (restricted according to the filters
        // received). Again, not showed here and not using async for now either.
        var report = new ReportDocument();
        report.Load(Server.MapPath("~/Reports/StatementReport.rpt"));
        report.SetDataSource(dataset);
        report.SetParameterValue("pUnitPrice", unitPrice);
        var stream = report.ExportToStream(
            ExportFormatType.PortableDocFormat
        );
        return File(stream, "application/pdf");
    }
}
```

Notes:

* The `ReportDocument` is defined in the `CrystalDecisions.CrystalReports.Engine` namespace.
* The `ExportFormatType` is defined in the `CrystalDecisions.Shared` namespace.

That was pretty much it.

It works as expected and I can use already existing code and assets almost as is.

Next, globalization!

#### References

This solution was heavily influenced by the following posts:

* [Crystal Reports in ASP.NET MVC (_coderguy123_ answer)](http://stackoverflow.com/a/2747571/215576)
* [Crystal Reports for Visual Studio 2010 Error](http://stackoverflow.com/q/4294762/215576)
