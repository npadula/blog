+++
title = "Faster Insert Performance with Entity Framework and SqlBulkCopy"
author = "Nicolas Padula"
date = "2020-04-07"
description = "Faster Insert Performance with Entity Framework and SqlBulkCopy"
tags = [
	"ef",
]
+++

While Entity Framework is a great ORM tool, sometimes it falls short of the desired performance in operations involving large quantities of inserts. Since EF doesn't support batch operations, the best approach in these cases is to do batch insert manually. 

SqlBulkCopy provides a straightforward way to do this. All you have to do is build a DataTable out of the entities you want to insert, pass it to SqlBulkCopy and it's done.

However, in a real world application your persistence logic will probably involve a more complex scenario, and you will have to mix EF and batch logic in the same transaction.

At work, I recently encountered a situation where I had to delete some records, insert some parent records, and then batch insert many child records, and it all needed to be wrapped in the same transaction in case something went wrong in the middle.

```aspnet
    public void SaveEntities(IEnumerable<MyEntity> entities, IEnumerable<MyEntity> entitiesToDelete)
            {
    			//this.dataContext is injected via DI, so we use it's connection string to open a new, different connection
                var connectionString = this.dataContext.Database.Connection.ConnectionString;
                var connection = (SqlConnection)this.dataContext.Database.Connection;
                var workspace = ((IObjectContextAdapter)this.dataContext).ObjectContext.MetadataWorkspace;
                var entityConnection = new EntityConnection(workspace, connection);
    
    			//Wrap everything inside a transaction
                using (var scope = Helpers.CreateTransactionScope())
                {
                    try
                    {
    
    
                        if (connection.State == ConnectionState.Closed)
                            connection.Open();
    
    					//Since this.dataContext is injected via DI, we cannot use that instance because it already owns a connection
    					//so we create a new instance and pass it an existing connection
                        var bulkContext = new MyDataContext(entityConnection); //Not the same as this.dataContext !!
    
    					
    					//Delete records
                        if(entitiesToDelete != null)
                        {
                            foreach (var a in entitiesToDelete)
                                bulkContext.MyEntities.Remove(a);
                        }
                        bulkContext.SaveChanges();
    
    						
                            foreach (var e in entities)
                            {
    							//Pass the connection used for bulkContext to SqlBulkCopy
                                var sqlBulkCopy = new SqlBulkCopy(connection);
    							/*
    								Do some stuff and insert some parent records
    							*/
    							
                                bulkContext.SaveChanges();
    
    
    
    
                                DataTable table = Helpers.BuildTable(aptusFromAssignment);
    
    
    
                                sqlBulkCopy.DestinationTableName = "MyEntitiesTable";
                                sqlBulkCopy.ColumnMappings.Add("SomeId", "SomeId");
                                sqlBulkCopy.ColumnMappings.Add("SomeValue", "SomeValue");
    
                                sqlBulkCopy.WriteToServer(table);
                            }
    
                        
    										//Commit transaction and close connection
                        scope.Complete();
                        connection.Close();
                    }
                    catch (Exception ex)
                    {
                        connection.Close();
                        throw ex;
                    }
                }
    
            }
```

Note that, while we are passing the same connection to the EF context and SqlBulkCopy, EF Context needs the connection to be an EntityConnection, while SqlBulkCopy just needs a plain SqlConnection.

The methods for building the table and creating the transaction scope are here:
```
    public class Helpers
        {
    				//Takes a collection of entities and build the datatable that will be inserted
            public static DataTable BuildTable(ICollection<MyEntity> entities)
            {
                var result = new DataTable();
    
    						//Set up the appropiate columns of the table
                result.Columns.Add("SomeId", typeof(int));
                result.Columns.Add("SomeValue", typeof(decimal));
    
    
                foreach (var e in entities)
                {
                    result.Rows.Add(e.SomeId, e.SomeValue);
                }
    
    
                return result;
            }
    
    
            public static TransactionScope CreateTransactionScope()
            {
                var transactionOptions = new TransactionOptions();
                transactionOptions.IsolationLevel = System.Transactions.IsolationLevel.ReadCommitted;
                transactionOptions.Timeout = TransactionManager.MaximumTimeout;
                return new TransactionScope(TransactionScopeOption.Required, transactionOptions);
            }
        }
        ```