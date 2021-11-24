# data
 Source = Excel.Workbook(File.Contents("C:\Users\91957\Downloads\global_superstore_2016.xlsx"), null, true),
    Orders_Sheet = Source{[Item="Orders",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(Orders_Sheet, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Row ID", Int64.Type}, {"Order ID", type text}, {"Order Date", type date}, {"Ship Date", type date}, {"Ship Mode", type text}, {"Customer ID", type text}, {"Customer Name", type text}, {"Segment", type text}, {"Postal Code", Int64.Type}, {"City", type text}, {"State", type text}, {"Country", type text}, {"Region", type text}, {"Market", type text}, {"Product ID", type text}, {"Category", type text}, {"Sub-Category", type text}, {"Product Name", type text}, {"Sales", type number}, {"Quantity", Int64.Type}, {"Discount", type number}, {"Profit", type number}, {"Shipping Cost", type number}, {"Order Priority", type text}}),
    #"removed row id, order id, shipmode" = Table.RemoveColumns(#"Changed Type",{"Row ID", "Order ID", "Ship Mode", "Customer ID", "Ship Date", "Postal Code", "Product ID", "Product Name", "Discount", "Order Priority"}),
    #"moved cust name to right" = Table.ReorderColumns(#"removed row id, order id, shipmode",{"Customer Name", "Order Date", "Segment", "City", "State", "Country", "Region", "Market", "Category", "Sub-Category", "Sales", "Quantity", "Profit", "Shipping Cost"}),
    #"Removed market, segment column" = Table.RemoveColumns(#"moved cust name to right",{"Market", "Segment"}),
    #"Split cust name" = Table.SplitColumn(#"Removed market, segment column", "Customer Name", Splitter.SplitTextByEachDelimiter({" "}, QuoteStyle.Csv, false), {"Customer Name.1", "Customer Name.2"}),
    #"renamed as first name" = Table.TransformColumnTypes(#"Split cust name",{{"Customer Name.1", type text}, {"Customer Name.2", type text}}),
    #"renamed as last name" = Table.RenameColumns(#"renamed as first name",{{"Customer Name.1", "First name"}, {"Customer Name.2", "Last name"}}),
    #"added ordered year" = Table.AddColumn(#"renamed as last name", "Year", each Date.Year([Order Date]), Int64.Type),
    #"Removed city" = Table.RemoveColumns(#"added ordered year",{"City"}),
    #"Rounded Up shipping cost" = Table.TransformColumns(#"Removed city",{{"Shipping Cost", Number.RoundUp, Int64.Type}}),
    #"Uppercased first name" = Table.TransformColumns(#"Rounded Up shipping cost",{{"First name", Text.Upper, type text}}),
    #"Capitalized first letter in first name" = Table.TransformColumns(#"Uppercased first name",{{"First name", Text.Proper, type text}}),
    #"Duplicated order date column" = Table.DuplicateColumn(#"Capitalized first letter in first name", "Order Date", "Order Date - Copy"),
    #"Calculated Quarter of ordered column" = Table.TransformColumns(#"Duplicated order date column",{{"Order Date - Copy", Date.QuarterOfYear, Int64.Type}}),
    #"renamed duplicate order date to quarter" = Table.RenameColumns(#"Calculated Quarter of ordered column",{{"Order Date - Copy", "Quarter"}})
in
    #"renamed duplicate order date to quarter"
