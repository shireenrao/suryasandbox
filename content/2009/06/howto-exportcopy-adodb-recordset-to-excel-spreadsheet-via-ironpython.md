Title: HOWTO export/copy ADODB recordset to excel spreadsheet via IronPython
Date: 2009-06-25 23:19
Author: Surya
Category: Rant
Slug: howto-exportcopy-adodb-recordset-to-excel-spreadsheet-via-ironpython
Status: published

    :::python
    """
    Export or Copy ADODB Recordset to Excel Spreadsheet
    @Author Surya Nyayapati
    @Date Jun-25-2009
    """
     
    import clr
     
    from System.Reflection import Assembly
    from System.Reflection import Missing
     
    #From C:\WINDOWS\assembly Copied the Display name for ADODB
    clr.AddReferenceByName('ADODB, Version=7.0.3300.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a')
    from ADODB import ConnectionClass, RecordsetClass, CursorLocationEnum
     
    connString = "DSN=myDsn;Uid=myUsername;Pwd=myPassword;"
     
    conn = ConnectionClass()
    rs = RecordsetClass()
    conn.Open(connString)
     
    conn.CursorLocation = CursorLocationEnum.adUseClient
    query = "SELECT * FROM user_group" # Any SELECT SQL
    rs = conn.Execute(query)
    """
    #To iterate over Recordset
    while rs.EOF <> True:   
        print "%s - %s" %(rs[0].Item[0].Name.ToString(),rs[0].Item[0].Value.ToString())
        print "%s - %s" %(rs[0].Item[1].Name.ToString(),rs[0].Item[1].Value.ToString()) 
            # ...
        rs.MoveNext()
    """ 
     
    #Start a New Excel Instance
    VSTOpath = "C:\\Program Files\\Microsoft Visual Studio 9.0\\Visual Studio Tools for Office\\PIA\\Office11\\"
     
    #Load and add reference to Excel dll
    excelAssemblyPath = "%sMicrosoft.Office.Interop.Excel.dll" % VSTOpath
    excelAssembly = Assembly.LoadFile(excelAssemblyPath)
    clr.AddReference(excelAssembly)
    from Microsoft.Office.Interop import Excel
     
    import System.IO.Directory
    excelApp = Excel.ApplicationClass()   
    excelApp.DefaultFilePath = System.IO.Directory.GetCurrentDirectory()
    excelApp.Visible = True
    excelApp.ScreenUpdating = False;
    excelApp.DisplayAlerts = False   
    excelApp.UserControl = False;
    try:
        book1 = excelApp.Workbooks.Add(Excel.XlWBATemplate.xlWBATWorksheet)
        sheet1 = book1.ActiveSheet
     
            headRange = sheet1.Range("A1", "D1")    
     
        #Python does not provide built-in support for multi-dimensional arrays
        from System import Array
        array2D = Array.CreateInstance(object, 2, 4)
        array2D[0,0] = "HEADER 1"
        array2D[0,1] = "HEADER 2"
        array2D[0,2] = "HEADER 3"
        array2D[0,3] = "HEADER 4"
     
        headRange.Value2 = array2D
        headRange.Font.Bold = True
        sheet1.Range("A2").CopyFromRecordset(rs, Missing.Value, Missing.Value)
     
    except StandardError, (ErrorMessage):
        print "something wrong (%s)" % ErrorMessage
     
    finally:
        excelApp.ScreenUpdating = True;
            excelApp.DisplayAlerts = True;
            excelApp.UserControl = True;
        #excelApp.Quit()
