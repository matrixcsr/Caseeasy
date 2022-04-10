Project Description
-------------
The Database and account management system is aimed to chart the data in a bar chart manner in an HTML page, the data will be shaped according to the year selected by the user, the user can further divide the selected data in Quarters, Months and weeks for the selected year. The system also allows to list all the Database metadata information like Id, Name, Size(in MB), Last access date, database creation date in a tabular format. The user can input custom parameters to search the databse by name for the desired size in a selected time time(Date) range. The data presented in the tabular form is sortable by every attribute, and also has the option for pagination limit. The System will also load a list of locked accounts in tabular form, the list data is supplied from an Api provided by Caseeasy.
### Table of Contents-
- Features
- Technologies Implemented
- Installation 
- Functioning of code
	- 	Client
		- 	Layout.cshtml
		- 	Index.cshtml
			- 	Graph Component
			-	List/Search DB
		- 	LockedAccounts.cshtml
				- GetLockedAccounts
	- 	Backend
		- 	Api Controller
		- 	Db1Controller.cs
				- GetDbCount()
			- 	SortByMonth()
			- 	SortByWeek()
			- 	SortByQuarter()
			- 	GetDbList()
		- 	LockedAccounts.cs
			- GetLockedAccounts()
		-	 Data Methods
			- 	SetAccessDate()
			- 	WeekCount()
			- 	QuarterCount()
			- 	QuarterLabeller()
			- 	WeekLabeller()
			- 	Monthlabeller()
		- 	HomeController.cs





### Features
**Dashboard**
- Chart Historical Data on graphs using  [Chart.js](https://www.chartjs.org/ "Chart.js").
- List the Database information (Id, Name, Size, Creation date, Last Access date).
**Locked Accounts**
	- List Locked Accounts and ability to unlock them.
### Technologies implemented
- C#
- AspNet MVC v5.2.7 (Framework net 6.0)
- SQL Server
- HTML5 / CSS, Bootstrap
- Javascript
- [Chart.js](https://www.chartjs.org/ "Chart.js")
- [DataTables](https://datatables.net/ "DataTables")
- RESTful web services
###**Installation**
- Run the final.sql script provided, on server to install the required Stored Procedures.
- Replacing the database connection string in the Controllers/**Db1Controller.cs** constructor
        public Db1Controller()
        {
            con.ConnectionString = "Replace this connection string";
            con.Open();
        }
- Changing the API port/domain to the server it is hosted on in Views/Home/**Index.cshtml**.

                $.ajax({
				   //Change the domain/port for every async api call.
                    url: "http://localhost:5027/api/DBAPI/Week",   
                    type: "POST",
                    contentType: "application/json",
                    },
                    error: function (err) {
                        alert("Something went wrong!");
                    },
                })

- Additional Visual studio libraries like Newtonsoft.Json might be required for serializing data*.

**Functioning of Code**
=============
**_Layout.cshtml**
-------------

- All the external JS or CDN scripts are loaded in Views/Shared/**_Layout.cshtml**, the Layout file also acts as the template for all the views inside Views/Home/*****
	The **_Layout.cshtml** also contains the Navbar which is displayed on top of every view.
- Layout file makes an async call to the api in the backend to display total number of databases in the navbar

                     $.ajax({
					 //Calls the api for loading the total database count.
                    url: "http://localhost:5027/api/DBAPI/DbCount",
                    type: "POST",
                    contentType: "application/json",
                    data: JSON.stringify(),
                    dataType: "json",
                    success: function (result) {
                        $("#db").append("Showing data on <b>"+result+"</b> databases"); //Appends text to Navabar
                    },
                    error: function (err) {
                        $("#db").append("Error Loading Db Count!");
                    },
                })

- The div below closing /header tag is responsible for rending all the views.
@RenderBody() loads the views.

**Index.cshtml**
-------------
###**Graph Component**
The Chart component is rendered  inside the canvas with id='mychart' at line 95
The Chart is renders and gets it X values(labels) at 

       
			 let barChart = new Chart(myChart, {
			  type: "bar",
                data: {
                    labels: [],                         //This is where the time labels or x-axis values goes.
                    datasets: [
                        {
                        },
                    ],
                }
The Chart is renders and gets it Y values(labels) at 


			
			let barChart = new Chart(myChart, {
			type: "bar",
                data: {
                    labels: [],                       
                    datasets: [
                        {
						label: "Database Count",
                            data: [],                     //This is where db_count or y-axis values goes.
                            backgroundColor: backgroundcolor
                        },
                    ],
                }`

- The Charts renders empty and the window.onload() is called which executes the Year() function, It makes a call to backend asking for Yearly data,.
            window.onload = function () {
                Year();                    
            };
	-The function expects an arugement typically a year(ie: 2021,2021...), if no arguements are specified, it will load data for all the years, if a year is specified, it will only load for the specified year.
	-After making the api call, it updates the data and label values and re-renders the chart using .update() method.
	                $.ajax({
                    url: "http://localhost:5027/api/DBAPI/Year",
                    type: "POST",
                    contentType: "application/json",
                    data: JSON.stringify(chartData),                
                    dataType: "json",
                    success: function (result) {
                        barChart.data.datasets[0].data = result.DbCount //updates the db count   
                        barChart.data.labels = result.DbYear;	//updates the labels on x-axis.
                        barChart.update()  //re- renders the chart.
						}

- On the top right, above the chart it generates a Select menu dynamically which lists a range of years from 2015 to current year.
            let dateDropdown = document.getElementById('date-dropdown');
            let currentYear = new Date().getFullYear();    
            let earliestYear = 2015;                       //Base year set to be 2015.
            let Option1 = document.createElement('option');
            Option1.text = "All";
            Option1.value = "0";
            dateDropdown.add(Option1);
            while (currentYear >= earliestYear) {        
                let dateOption = document.createElement('option');
                dateOption.text = currentYear;
                dateOption.value = currentYear;
                dateDropdown.add(dateOption);               
                currentYear -= 1;                              //decrements from currentYear on every iteration.
            }
	-The select tag has an onChange() method, which calls a function which check the option currently selected is a specific year or "All", if a specific year is selected, it will render 3 buttons on the bottom of the charts(Weekly,Monthly,Quarterly), if the option selected is "All" it won't render those buttons. 
	- "All" is supposed to render all the data year on year, thus breaking it down into timelines is not possible.

                       function GetSelectedTextValue(selectedOption) {
					   var selectedText = selectedOption.options[selectedOption.selectedIndex].innerHTML;
                    var selectedValue = selectedOption.value;       
                    console.log(selectedValue);
                    $("div #test").empty();                        
				//Will only create buttons if the value of the button is not "All"        
                    if (selectedValue > 0) {
					  
                        var element1 = document.createElement("button");   
                        var element2 = document.createElement("button");  
                        var element3 = document.createElement("button");
- The buttons created dynamically after selected a year, will have an onClick() method, which calls their respective method

                        element1.innerText = "Weekly";
						element1.value = selectedValue;
                        element1.className = "btn btn-light";
                        element1.name = "weekly";
                        element1.onclick = function () {
                            Week(selectedValue);
                        }             //Weekly button will call the Week method onClick.

                        element2.innerText = "Monthly";
                        element2.value = selectedValue;
                        element2.className = "btn btn-light";
                        element2.name = "Monthly";
                        element2.onclick = function () {
                            Month(selectedValue);
                        }            //Monthly button will call the Month method onClick.

                        element3.innerText = "Quarterly";
                        element3.value = selectedValue;
                        element3.className = "btn btn-light";
                        element3.name = "Quarterly";
                        element3.onclick = function () {
                            Quarter(selectedValue);
                        }           //Quarterly button will call the Quarter method onClick.

-The functions called onClick() are similar functioning to Year().

###**List/Search DB**
-The has a form and a table([Datatables](https://datatables.net/ "Datatables"))
- The form takes in user input for
	- Database name
	- Database size
	- Size greater than or less than specified.
	- Date range(from-till)

- The Datatable is rendered dynamically  by calling the .DataTable() method.
` var table = $("#table_id").DataTable();`
- The when the Search button is clicked,it passes the form data through an async api call, it empties the table and renders the data into the table.
-The datatable allows pagination, search functionality and sorting via any column.


**LockedAccounts.cshtml**
-------------
###**GetLockedAccounts**
- The component renders a datatable onLoad(), it makes call to an external api provided by Caseeasy, and renders the api data in a tabular form.
- The Datatable has an extra column, which houses a link for every row, upon clicking the link it re-directs to Caseeasy admin portal, where the admin will be able to unlock the account.

**Backend Code**
-------------
###**Api Controller**
-The Api Controller that is responsible for handling async calls from the Views can be found inside ./Controllers/**DBAPIController.cs**


    {

		[Route("DbCount")]
        [HttpPost]
        public string PostDbCount()
        {
            Db1Controller db = new Db1Controller();
			//Executes the GetDbCount() inside ./Db1Controller.cs
            string sJSONResponse = JsonConvert.SerializeObject(db.GetDbCount());
            return sJSONResponse;
        }



        [Route("Month")]
        [HttpPost]
        public string PostMonth(ChartValues data)
        {
            Db1Controller db = new Db1Controller();
			//Executes the SortByMonth() inside ./Db1Controller.cs
            string sJSONResponse = JsonConvert.SerializeObject(db.SortByMonth(data));
            return sJSONResponse;
        }

        [Route("Year")]
        [HttpPost]
        public string PostYear(ChartValues data)
        {
			
                Db1Controller db = new Db1Controller();
                int step = 0;
                string dateY = DateTime.Now.Year.ToString();
                int CurrentYear = int.Parse(dateY);
                List<int> count = new List<int>();
                List<int> yearLabel = new List<int>();
                int baseYear = 2000;
				//Will list only for the year selected
                if(data.year != 0)
                {
                count.Add(db.SortByYear(data.year));
                yearLabel.Add(data.year + step);
                }
				//Will list the data for all the years... (Selected "All" in the select menu)
            else
            {
                for (int i = baseYear; i <= CurrentYear; i++)
                                {

                                    if (db.SortByYear(i) != 0)
                                    {
                                        count.Add(db.SortByYear(i));
                                        yearLabel.Add(baseYear + step);
                                    }
                                    step++ ;
                                }
            }
                
                DisplayData dd = new DisplayData()
                {
                    DbCount = count,
                    DbYear = yearLabel
                };
                string sJSONResponse = JsonConvert.SerializeObject(dd);
                return sJSONResponse;

        }

        [Route("Week")]
        [HttpPost]
        public string PostWeek(ChartValues data)
        {
            Db1Controller db = new Db1Controller();
			//Executes the SortByWeek() inside ./Db1Controller.cs
            string sJSONResponse = JsonConvert.SerializeObject(db.SortByWeek(data));
            return sJSONResponse;

        }
        [Route("Quarter")]
        [HttpPost]
        public string PostQuarter(ChartValues data)
        {
            Db1Controller db = new Db1Controller();
			//Executes the SortByQuarter() inside ./Db1Controller.cs
            string sJSONResponse = JsonConvert.SerializeObject(db.SortByQuarter(data));
            return sJSONResponse;
        }
        [Route("SearchDB")]
        [HttpPost]
        public List<DatabaseList> PostSearchDB(FormData data)
        {
            Db1Controller db = new Db1Controller();
            List<DatabaseList> dbList = new List<DatabaseList>();
			//Executes the GetDbList() inside ./Db1Controller.cs
            dbList = db.GetDbList(data);
            
            return dbList;
        }

        [Route("AccountList")]
        [HttpPost]
        public List<AccountList> PostAccountList()
        {
            LockedAccounts la = new LockedAccounts();
            List<AccountList> AccountList = new List<AccountList>();
			//Executes the GetLockedAccounts() inside ./LockedAccounts.cs
            AccountList = la.GetLockedAccounts();
            return AccountList;
        }
    }

###**Db1Controller.cs**
- This Class is responsible for handling the data shaping and executing the Stored procedures inside the SQL server .
- The Constructor for this class is responsible for opening connection to server.
 
	    public Db1Controller()
		{
            con.ConnectionString = "Database connection String";
            con.Open();
        }
1. **GetDbCount()**
	The Function executes the Db_Count() SP and return a Dataset() with total db_count (excludes the system databases)
	 
       
	   public int GetDbCount()
	   {
            int count = 0;
            DataSet ds = new DataSet();
            cmd = new SqlCommand("Db_Count", con);          //Db_Count SP is executed to fetch the total Database count.
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.ExecuteNonQuery();
            SqlDataAdapter da = new SqlDataAdapter(cmd);
            da.Fill(ds);
            foreach (DataRow dr in ds.Tables[0].Rows)
            {
                count = Convert.ToInt32(dr["db_count"]);    // returned row(s) are then stored to the int count.
            }

            con.Close();            //Connection closed.
            return count;           //Db Count is returned.

        }
2 **SortByMonth()**
- The Function executes the GetMonthCounts SP inside the SQL Server
- The Function is responsible for splitting the data down to Monthly level for a specified year.
- IT takes in an arguement, which is a specific year, that is selected from the drop down menu, the arguement is Year number (ie: 2020)
- The function calls another function of class DataMethods.cs, it is used to change the index number of month(2,3) to String Labels(Feb,Mar).
        
		public DisplayData SortByMonth(ChartValues data) {
            DataMethods dm = new DataMethods();
            DataSet ds = new DataSet();
            List<int> DbCount = new List<int>();                        
            List<String> DbLabel = new List<String>();
            string? Label;
            cmd = new SqlCommand("GetMonthCounts", con);        //GetMonthCounts SP is executed.
            cmd.CommandType = CommandType.StoredProcedure;
            SqlDataAdapter da = new SqlDataAdapter(cmd);
            cmd.ExecuteNonQuery();
            da.Fill(ds);
             foreach (DataRow dr in ds.Tables[0].Rows)
            {

                if(Convert.ToInt32(dr["y"]) == data.year)
                {
                    int Count = Convert.ToInt32(dr["tally"]);       //tally refers to count returned for the particular year.
                    DbCount.Add(Count);

                Label = dm.Monthlabeller(Convert.ToInt32(dr["m"]), Convert.ToInt32(dr["y"])); //Transforms data into label
                DbLabel.Add(Label);
                }
             
            }
            DisplayData dd = new DisplayData()          //Both the lists are addedd to the model.
            {
                DbCount = DbCount,                  
                DbMonth = DbLabel
            };

            con.Close();
            return dd;                  //Model returned back to api call.
        }
3 **SortByWeek()**
- The Function executes the GetWeekCounts SP inside the SQL Server
- The Function is responsible for splitting the data down to Weekly level for a specified year.
- IT takes in an arguement, which is a specific year, that is selected from the drop down menu, the arguement is Year number (ie: 2020)
- The function calls another function WeekLabeller() of class DataMethods.cs, It takes in 2 arguements, day of month and month index. Using this info, it converts number into information like (12,4) to "Apr/Q2" string.
- There maybe times where we will have multiple items with the same Label "Apr/Q2", So another function WeekCount() of DataMethods.cs is called to take unique labels and take sum of a particular repeting label and write the unqiue label "Apr/Q2" with sum(3,5,7) to another empty list.
        public DisplayData SortByWeek(ChartValues data)     //Data refers to the year passed from client side.
        {
            DataMethods dm = new DataMethods();
            DataSet ds = new DataSet();
            List<int> DbCount = new List<int>();
            List<String> DbLabel = new List<String>();
            cmd = new SqlCommand("GetWeekCounts", con); 
            cmd.CommandType = CommandType.StoredProcedure;
            SqlDataAdapter da = new SqlDataAdapter(cmd);
            cmd.ExecuteNonQuery();
            da.Fill(ds);
            string Label;
            int Count;

            //Gets the db counts for the corresponding day,month and year, additional label processing is done later.
            foreach (DataRow dr in ds.Tables[0].Rows)
            {
                if (Convert.ToInt32(dr["y"]) == data.year)
                {
                    Count = Convert.ToInt32(dr["tally"]);
                    DbCount.Add(Count);
                    //refer to the method documentation in class for more info.
                    Label = dm.WeekLabeller(Convert.ToInt32(dr["day_of_creation"]), Convert.ToInt32(dr["m"]), Convert.ToInt32(dr["y"]), Convert.ToInt32(dr["tally"]));
                    DbLabel.Add(Label);
                }

            }

            List<String> DbLabel1 = new List<String>();
            List<int> DbCount1 = new List<int>();
            for (int i = 0; i < DbLabel.Count; i++)
            {
                if (!DbLabel1.Contains(DbLabel[i]))
                {
				//Eliminates duplicate labels, and sums up count of duplicate labels.
                    Count = dm.WeekCount(DbCount, DbLabel,DbLabel[i]);
                    DbCount1.Add(Count);
                    DbLabel1.Add(DbLabel[i]);
                }
            }

            //adding the second summed up list to model.
            DisplayData dd = new DisplayData()
            {
                DbCount = DbCount1,
                DbMonth = DbLabel1
            };

            return dd;
        }
		
4 **SortByQuarter()**
- The Function executes the GetQuarterCounts SP inside the SQL Server
- The Function is responsible for splitting the data down to Quarterly level for a specified year.
- IT takes in an arguement, which is a specific year, that is selected from the drop down menu, the arguement is Year number (ie: 2020)
- The function calls another function QuarterLabeller() of class DataMethods.cs, It takes in 3 arguements, day of month and month, year index. Using this info, it converts number into information like (12,4,2020) to "2020-Apr/Q2" string.
- There maybe times where we will have multiple items with the same Label "Q2-2020", So another function QuarterCount() of DataMethods.cs is called to take unique labels and take sum of a particular repeting label and write the unqiue label "Apr/Q2" with sum(3,5,7) to another empty list.

       public DisplayData SortByQuarter(ChartValues data)
	   {
            DataMethods dm = new DataMethods();
            DataSet ds = new DataSet();
            List<int> DbCount = new List<int>();
            List<String> DbLabel = new List<String>();
			//Executes the GetQuarterCounts SP on the server.
            cmd = new SqlCommand("GetQuarterCounts", con);     
            cmd.CommandType = CommandType.StoredProcedure;
            SqlDataAdapter da = new SqlDataAdapter(cmd);
            cmd.ExecuteNonQuery();
            da.Fill(ds);
            int Count;
            String Label;

            //Returns data in month,year and db_count(tally) format, data shaping is done later.
            foreach (DataRow dr in ds.Tables[0].Rows)
            {
                if (Convert.ToInt32(dr["y"]) == data.year)
                {
                Count = Convert.ToInt32(dr["tally"]);
                DbCount.Add(Count);
                Label = dm.QuarterLabeller(Convert.ToInt32(dr["m"]), Convert.ToInt32(dr["y"]));
                DbLabel.Add(Label);
                }

            }

            List<String> DbLabel1 = new List<String>();
            List<int> DbCount1 = new List<int>();

            for (int i = 0; i < DbLabel.Count; i++)
            {
                if (!DbLabel1.Contains(DbLabel[i]))
                {
                    Count = dm.QuarterCount(DbCount, DbLabel, DbLabel[i]);
                    DbCount1.Add(Count);
                    DbLabel1.Add(DbLabel[i]);
                }
            }

            //Adding second list to the model.
            DisplayData dd = new DisplayData()
            {
                DbCount = DbCount1,
                DbMonth = DbLabel1
            };

            return dd;
        }
		
5 **SortByYear()**
- The Function executes the GetYearCounts SP inside the SQL Server
- This Function is executed repeatedly if the data is needed to be fetched foe "All" option or once for a given year.
- The repeted execution of this function is done in the DB API controller.

        public int SortByYear(int year)
        {
            int YearCount = 0;
            DataSet ds = new DataSet();
            String CurrentYear = DateTime.Now.Year.ToString();
            int a = int.Parse(CurrentYear);
            cmd = new SqlCommand("GetYearCounts", con);
            cmd.CommandType = CommandType.StoredProcedure;
            SqlDataAdapter da = new SqlDataAdapter(cmd);
            cmd.Parameters.AddWithValue("@year", SqlDbType.Int).Value = year;
            cmd.Parameters.AddWithValue("@rowCount", SqlDbType.Int).Value = 1; //Only for keeping a check on empty sets.
            cmd.ExecuteNonQuery();
            int count = Convert.ToInt32(cmd.Parameters["@rowCount"].Value);
            da.Fill(ds);
            foreach (DataRow dr in ds.Tables[0].Rows)
            {
                if (count > 0)
                {
                    YearCount = Convert.ToInt32(dr["tally"]);
                }

            }

            return YearCount;

        }

6 **GetDbList()**
- The Function executes the GetDbList SP inside the SQL Server.
- The Function takes in all the form data as arguement, passed in a model.
- The SP return 5 attributes
	1. Database Id
	2. Database Name
	3. Database last access date
	4. Database Creation date

- The Database last access date is replaced by the date supplied from an external API provided by Caseeasy.
- To replace the date, DataMethods.cs method is called, where the database name is compared and access date is changed.

 
        public List<DatabaseList> GetDbList(FormData data)  
		{

            List<DatabaseList> dbList = new List<DatabaseList>();
            DataMethods dm = new DataMethods();
            ServicePointManager.Expect100Continue = true;
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
            //Calls the api from Caseeasy.
            string json = new WebClient().DownloadString("https://caseeasymicroservices.azurewebsites.net/api/KPIService");
            //Deserializing the String to Json.
            var dbList1 = JsonConvert.DeserializeObject<List<DatabaseList>>(json);
            DataSet ds = new DataSet();
            cmd = new SqlCommand("GetDbList", con); //Execute GetDbList SP on SQL server.
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@DbName", data.name);
            cmd.Parameters.AddWithValue("@Symbol", data.symbol);
            cmd.Parameters.AddWithValue("@Begindate", data.beginDate);
            cmd.Parameters.AddWithValue("@Enddate", data.endDate);
            cmd.Parameters.AddWithValue("@Size", data.size);

            SqlDataAdapter da = new SqlDataAdapter(cmd);
            da.Fill(ds);

            //Returns the database list and other meta data.
            foreach (DataRow dr in ds.Tables[0].Rows)
            {

                dbList.Add(new DatabaseList
                {
                    id = Convert.ToInt32(dr["database_id"]),
                    name = dr["name"].ToString(),
					//Date is changed with the one provided by Caseeasy API
                    accessDate = Convert.ToDateTime(dm.SetAccessDate(dr["name"].ToString(), dbList1)).ToString("dd/MM/yyyy"),
                    createDate = Convert.ToDateTime(dr["Create_Date"]).ToString("dd/MM/yyyy"),
                    size = Convert.ToInt32(dr["size"])

                });

            }

            return dbList;

        }
		
###**LockedAccounts.cs**
-------------
1 **GetLockedAccounts()**
- The Function fetches the API provided by Caseeasy, it deserializes it and stores it in a list.
- It loops through the list structures it in a proper way, and the list is returned back to the frontend via API.
        public List<AccountList> GetLockedAccounts()
		{
            List<AccountList> AccountList = new List<AccountList>();
            ServicePointManager.Expect100Continue = true;
            ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;

            //Calling the api provided by Caseeasy and deserializing it to json.
            string json = new WebClient().DownloadString("https://caseeasymicroservices.azurewebsites.net/api/AccountManagerService");
            var lockedAccounts = JsonConvert.DeserializeObject<List<AccountList>>(json);

            //Iterating through all the objects in json and writing it to a list.
            for (int i = 0; i < lockedAccounts.Count; i++)
            {
                AccountList.Add(new AccountList
                {
                    id = lockedAccounts[i].id,
                    name = lockedAccounts[i].name,
                    siteId = lockedAccounts[i].siteId,
                    url = lockedAccounts[i].url
                }) ;

            }

            return AccountList;
        }

###**Controllers/Methods/DataMethods.cs**
- This class is only made for labelling the data to present it better on chart.
- There are also methods for summing up data, to give it a prope shape and avoid repetitions.
-------------
1 **SetAccessDate()**
	- The function is used to set/replace last access date from database with the one provided by  Caseeasy API
- The function compares the database names with api database names and replaces the dates accordingly.

        public String SetAccessDate(String DbName, List<DatabaseList> ApiResult)
        {

            String Accessdate = DateTime.UtcNow.ToString();
            
            for (int i = 0; i < ApiResult.Count; i++)
            {
                if (ApiResult[i].name == DbName)
                {
                    Accessdate = ApiResult[i].accessDate.ToString();
                    break;
                }

            }
            return Accessdate;
        }
2 **WeekCount()**
- This Function is used to only keep unqiue values of labels, and sum up the database counts against the index of repeated label, the end out will be one unqiue label of a specific, and sum total of integers where the same labels was repeating.
        public int WeekCount(List<int> DbCount,List<String> DbLabel, String label)
        {
            int sum = 0;
            for (int i = 0; i < DbLabel.Count; i++)
            {
                
                if(DbLabel[i] == label)
                {
                    sum += DbCount[i];
                }
            }
                return sum;
        }

3 **QuarterCount()**
 - This Function is used to only keep unqiue values of labels, and sum up the database counts against the index of repeated label, the end out will be one unqiue label of a specific, and sum total of integers where the same labels was repeating (same as WeekCount).
       
		 public int QuarterCount(List<int> DbCount, List<String> DbLabel, String label){
            int sum = 0;
            for (int i = 0; i < DbLabel.Count; i++)
            {

                if (DbLabel[i] == label)
                {
                    sum += DbCount[i];
                }
            }
            return sum;

        }

4 **QuarterLabeller()**
- The Function takes in month and year integers, and converts the data to String label.
        public String QuarterLabeller(int Month, int Year){
            String label = null;
            if (Month > 0 && Month <=3)
            {
                label = Year + "-Q1";
            }
            else if (Month >= 4 && Month <= 6)
            {
                label = Year + "-Q2";
            }
            else if (Month >= 6 && Month <= 9)
            {
                label = Year + "-Q3";
            }
            else if (Month >= 9 && Month <= 12)
            {
                label = Year + "-Q4";
            }
            return label;
        }

5 **WeekLabeller()**
- The Function takes in Day, Month, Year,<s>Count</s> integers as input and shapes the data for week labels.

         public String WeekLabeller(int Day, int Month, int Year, int Count){
            String label = "";
            if (Month == 1)                     
            {
                if(Day > 0 && Day <= 7)
                {
                    label = "Jan/Q1-" + Year;
                }
                else if(Day>7 && Day <= 14)
                {
                    label = "Jan/Q2-" + Year;
                }
                else if(Day>14 && Day <= 21)
                {
                    label = "Jan/Q3-" + Year;
                }
                else if (Day > 21)
                {
                    label = "Jan/Q4-" + Year;
                }
				//And a long else if ladder for each month.

6 **Monthlabeller()**
- This Function takes in Month and Year input, and converts those integers into String labels.
        public String Monthlabeller(int Month, int Year)
        {
            String label = "";
            if(Month == 1)
            {
                label = "Jan-" + Year;
            }
            else if(Month == 2)
            {
                label = "Feb-" + Year;
            }
            else if(Month == 3)
            {
                label = "Mar-" + Year;
            }
            else if (Month == 4)
            {
                label = "Apr-" + Year;
            }
            else if (Month == 5)
            {
                label = "May-" + Year;
            }
            else if (Month == 6)
            {
                label = "Jun-" + Year;
            }
            else if (Month == 7)
            {
                label = "Jul-" + Year;
            }
            else if (Month == 8)
            {
                label = "Aug-" + Year;
            }
            else if (Month == 9)
            {
                label = "Sept-" + Year;
            }
            else if (Month == 10)
            {
                label = "Oct-" + Year;
            }
            else if (Month == 11)
            {
                label = "Nov-" + Year;
            }
            else if (Month == 12)
            {
                label = "Dec-" + Year;
            }

            return label;
        }

###**HomeController.cs**
-------------
- This Controller is responsible for handling routes. It will display a specific view for the route visited.
-Available Routes...
- 	 **"/" **- Route for the main dashboard(Charts/SearchDb) -loads Index.cshtml view
- 	 **"/Home/LockedAccounts"** - Route for Locked Accounts - loads LockedAccounts.cshtml view



        public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger)
        {
		_logger = logger;
        }

        public IActionResult Index()
        {
		return View();     //will load Index.cshtml
        }
        public IActionResult LockedAccounts()
        {

            return View();     //will load LockedAccounts.cshtml
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
        }
    }


-Documented by [Rahul Soni](https://twitter.com/Matrix_Csr "Rahul Soni")
- Team Tech **S.T.A.R**
###End
