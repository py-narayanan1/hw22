.. code:: ipython3

    import openpyxl as pxl

.. code:: ipython3

    import pandas as pd

.. code:: ipython3

    from pathlib import Path

The input file even if exported to a different folder structure will
conform to its structure since the internal folder structure of input
and logs is the same

.. code:: ipython3

    input_file = Path.cwd().joinpath("input").joinpath("logs")

.. code:: ipython3

    input_file




.. parsed-literal::

    WindowsPath('C:/Users/narayanan/Downloads/hw22_visweswaran/hw22_visweswaran/notebooks/input/logs')



The BCM type files are taken up here

.. code:: ipython3

    for f in input_file.rglob('BCM*.csv'):
        print(f"filename = {f.name}, stem = {f.stem}, suffix = {f.suffix}")


.. parsed-literal::

    filename = BCM-E-tCenter-Deep.csv, stem = BCM-E-tCenter-Deep, suffix = .csv
    filename = BCM-E-tCenter-Medium.csv, stem = BCM-E-tCenter-Medium, suffix = .csv
    filename = BCM-E-tCenter-Shallow.csv, stem = BCM-E-tCenter-Shallow, suffix = .csv
    filename = BCM-E-tLeft-Deep.csv, stem = BCM-E-tLeft-Deep, suffix = .csv
    filename = BCM-E-tLeft-Medium.csv, stem = BCM-E-tLeft-Medium, suffix = .csv
    filename = BCM-E-tRight-Deep.csv, stem = BCM-E-tRight-Deep, suffix = .csv
    filename = BCM-E-tRight-Medium.csv, stem = BCM-E-tRight-Medium, suffix = .csv
    filename = BCM-N-tCenter-Deep.csv, stem = BCM-N-tCenter-Deep, suffix = .csv
    filename = BCM-N-tCenter-Medium.csv, stem = BCM-N-tCenter-Medium, suffix = .csv
    filename = BCM-N-tLeft-Deep.csv, stem = BCM-N-tLeft-Deep, suffix = .csv
    filename = BCM-N-tLeft-Medium.csv, stem = BCM-N-tLeft-Medium, suffix = .csv
    filename = BCM-N-tRight-Deep.csv, stem = BCM-N-tRight-Deep, suffix = .csv
    filename = BCM-N-tRight-Medium.csv, stem = BCM-N-tRight-Medium, suffix = .csv
    filename = BCM-N-tRight-Shallow.csv, stem = BCM-N-tRight-Shallow, suffix = .csv
    

We create an empty dictionary where the key will be the stem of the file
and the value will be the datframe that has been produced off from the
csv file imported for each csv

.. code:: ipython3

    dfs={}

The rglob recursively scans through all the csv files and puts them in a
format that has been mentioned above

.. code:: ipython3

    for f in input_file.rglob('BCM*.csv'):
        fstem = f.stem
        df = pd.read_csv(f,header=None)
        dfs[fstem] = df

In the output folder inside the notebooks folder , the BCM.xlsx file
which has been manually created is accesses and then the dataframe is
looped over , where in the datatypes of each column is updated and the
headers are also set as per the format expected , the good thing about
excelwriter is , it doesnt matter how many times we run it , if the
syntax remains the same, no new excel sheets are created. in the
to_excel formula , we pass in the sheet name as the key of the
dictionary and which in turn is the stem name of each file

.. code:: ipython3

    with pd.ExcelWriter('output/BCM.xlsx') as writer:
        for sheetname,sheetdata in dfs.items():
                sheetdata = sheetdata.astype({0: 'datetime64',1: 'string',2:'float64'})
                sheetdata.to_excel(writer,sheet_name=sheetname,index=False,header=['datetime','scale','temperature'])

The pxl is the shortform of openpyxl module which is then loaded on to
the workbook

.. code:: ipython3

    wb = pxl.load_workbook("output/BCM.xlsx")

We imported mean for faster processing of the data

.. code:: ipython3

    from statistics import mean

This is the process of filling in each sheet created in the excel file

.. code:: ipython3

    """loop through all the sheetnames in the target excel file"""  
    for x in wb.sheetnames:
        """  Acces the worksheet with that particular sheetname"""  
        ws = wb[x]
        """ Fill in all the cells as have been asked for in the document since everyone of them has a common format , 
        it can be repeated"""  
        ws['G2'] = "min_temp"
        ws['G3'] = "max_temp"
        ws['G4'] = "mean_temp"
        ws['G6'] = "min_date"
        ws['G7'] = "max_date"
        """  date cells are the cells which have been created to access just the dates"""  
        date_cells = []
        """  similarily for temperature cells"""  
        temperature_cells = []
        """  the row pack is a packer that creates an iterable over the rows from the 2nd to max length of the rows and it moves along
        the first column for the dates"""  
        for row_pack in ws['A2':'A'+str(ws.max_row)]:
            """  loops through the iterable """  
            for row in row_pack:
                date_cells.append(row.value)
        """  Finds the minimum of all the dates"""  
        ws['H6'] = min(date_cells)
        """  Finds the maximum of all the dates"""  
        ws['H7'] = max(date_cells)
        """  Similar method for the temperature values"""  
        for row_pack in ws['C2':'C'+str(ws.max_row)]:
            for row in row_pack:
                temperature_cells.append(row.value)
        ws['H2'] = min(temperature_cells)
        ws['H3'] = max(temperature_cells)
        ws['H4'] = mean(temperature_cells)
    """  Finally we save the entire updated excel file with all its changes"""  
    wb.save('output/BCM.xlsx')   

Hacker

Now since we have a different approach for it , now I have created 2
folders called logs_hacker in the input folder inside the notebooks
folder , where the file names of the excel csvs have been changed to a
different filename , so that for each filename and different output
excel file is created which goes into a folder named output_hacker.

.. code:: ipython3

    """   A set of files is created to avoid any repetition"""  
    set_of_files = set()
    dire = Path('input/logs_hacker')
    """  A new dictionary is created for creating a stem for each new type of file name"""  
    dfs_hacker = {}
    for f in dire.glob('*.csv'):
        fstem = f.stem
        df_hacker = pd.read_csv(f,header=None)
        dfs_hacker[fstem] = df_hacker
        """  We just access the first three letters the file to see if it is differnt"""  
        set_of_files.add(fstem[0:fstem.index('-')])
        

.. code:: ipython3

    """  looping through the set"""  
    for x in list(set_of_files):
        """  creating a workbook for each new filename"""  
        wb = pxl.Workbook()
        """  saving creates the file in the destination folder."""  
        wb.save('output_hacker/'+x+'.xlsx')
        """  creating a writer object to fill in the individual excel files with their excel sheets"""  
        with pd.ExcelWriter('output_hacker/'+x+'.xlsx') as writers:
          for sheetname,sheetdata in dfs_hacker.items():
            """  checks if the sheetname is same as the filename of the new excel file created. i.e. the first three letters,if
                yes , then creates a new sheet in that designated excel file for that kind of file and repeats the same."""  
            if sheetname[0:sheetname.index('-')] == x:
                sheetdata = sheetdata.astype({0: 'datetime64',1: 'string',2:'float64'})
                sheetdata.to_excel(writers,sheet_name=sheetname,index=False,header=['datetime','scale','temperature'])
            else:
                continue
       
        
        

