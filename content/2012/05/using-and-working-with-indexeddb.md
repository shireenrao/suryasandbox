Title: Using and working with IndexedDB
Date: 2012-05-17 12:32
Author: Surya
Category: Rant
Slug: using-and-working-with-indexeddb
Status: published
    
Below is a sample example code for IndexedDB  
    
    :::javascript
    //nsr is namespace
    (function (nsr, $, undefined) {
        //private variables
        nsr.myTvQ.indexedDB = nsr.myTvQ.indexedDB || {};
        nsr.myTvQ.indexedDB.db = null;
        nsr.myTvQ.indexedDB.version = "1.0";
         
        //saving webkitIndexedDB in global namespace
        if ('webkitIndexedDB' in window) {
            window.indexedDB = window.webkitIndexedDB;
            window.IDBTransaction = window.webkitIDBTransaction;
            window.IDBKeyRange = window.webkitIDBKeyRange;
            window.IDBCursor = window.webkitIDBCursor;
        }
        //global error handler
        nsr.myTvQ.indexedDB.onerror = function (e) {
            console.log("myError: ", e);        
        };
        // callback param in integer, -1 error, 0 exiting, 1 new
        nsr.myTvQ.indexedDB.Open = function (callback) {
            if (nsr.myTvQ.indexedDB.db) {
                if (callback) {
                    callback(0);//opened but present
                }
                return;
            }
            var request = indexedDB.open("myTvQDB", "comments bla blah database v1.0");
     
            request.onsuccess = function (e) {
                var v = nsr.myTvQ.indexedDB.version;
                nsr.myTvQ.indexedDB.db = e.target.result;
                var db = nsr.myTvQ.indexedDB.db;
                console.log('opened indexedDB myTvQDB v:' + db.version);            
     
                // We can only create Object stores in a setVersion transaction;
                // so if version not found update or create new db
                if (v != db.version) {
                    console.log('setting new v ',v, db.version);
                    var setVrequest = db.setVersion(v);
     
                    // onsuccess is the only place we can create Object Stores
                    setVrequest.onfailure = request.onfailure =  function (e) 
                    {        
                        console.log("myError: ", e); 
                        if (callback) {                        
                            callback(-1);
                        }
                    };
                    setVrequest.onsuccess = function (e) {
                        //create this at last, since update_time is stored in end
                        nsr.myTvQ.indexedDB.DropObjectStore("settings");
                        nsr.myTvQ.indexedDB.CreateSettingsStore();
     
                        nsr.myTvQ.indexedDB.DropObjectStore("listings");
                        nsr.myTvQ.indexedDB.listing.CreateListingsStore();         
                        if (callback) {
                            callback(1);//created                        
                        }
                    };
                } else {
                    if (callback) {
                        callback(0);//opened but present
                    }
                    console.log('object stores are present');
                }
            };
     
            request.onfailure =  function (e) 
            {        
                console.log("myError: ", e); 
                if (callback) {
                    callback(-1);//error
                }
            };
        }
         
        //helper methods
        nsr.myTvQ.indexedDB.Close = function(){
            try {
                var db = nsr.myTvQ.indexedDB.db;
                if (db) {
                    db.close();
                    nsr.myTvQ.indexedDB.db = null;
                    if (callback) {
                        callback(1);
                    }
                }   
            } catch (ex) {
                if (callback) {
                    callback(-1);
                }
                console.log("Exception in Close() - " + ex.message);
            }
        } 
        nsr.myTvQ.indexedDB.DeleteDatabase = function(callback){
            try {
                var db = nsr.myTvQ.indexedDB.db;
                if (db) {
                    db.close();
                }   
                var request =  window.indexedDB.deleteDatabase('myTvQDB');
                request.onsuccess = function (e) {
                    nsr.myTvQ.indexedDB.db = null;
                    if (callback) {
                        callback(1);
                    }
                    console.log('Database successfully deleted');
                };
                request.onfailure = function (e) 
                {        
                    console.log("myError: ", e); 
                    if (callback) {
                        callback(-1);
                    }
                };    
            } catch (ex) {
                if (callback) {
                    callback(-1);
                }
                console.log("Exception in DeleteDatabase() - " + ex.message);
            }
        }
         
        nsr.myTvQ.indexedDB.DropObjectStore = function (storeName) {
            var db = nsr.myTvQ.indexedDB.db;
            if (db.objectStoreNames.contains(storeName)) {
                db.deleteObjectStore(storeName);
                console.log('objectstore ' + storeName + ' dropped');
            }
        }
     
        nsr.myTvQ.indexedDB.ForceDropObjectStore = function (storeName) {
            var db = nsr.myTvQ.indexedDB.db;
            var setVrequest = db.setVersion(Math.random());
            setVrequest.onfailure = nsr.myTvQ.indexedDB.onerror;
            setVrequest.onsuccess = function (e) {
                console.log('db.version ', db.version);
                nsr.myTvQ.indexedDB.DropObjectStore(storeName);
            }
        };
         
        //can use to re-create empty ObjectStore
        nsr.myTvQ.indexedDB.ForceCreateObjectStore = function (storeName) {
            console.log('ForceCreateObjectStore', storeName);
            var db = nsr.myTvQ.indexedDB.db;
            var setVrequest = db.setVersion(Math.random());
            setVrequest.onfailure = nsr.myTvQ.indexedDB.onerror;
            setVrequest.onsuccess = function (e) {
                console.log('db.version ', db.version);
                nsr.myTvQ.indexedDB.DropObjectStore(storeName);
                switch(storeName){
                    case "listings":
                        nsr.myTvQ.indexedDB.listing.CreateListingsStore();
                        break;  
                    case "settings":
                        nsr.myTvQ.indexedDB.listing.CreateSettingsStore();
                        break;     
                    default:
                        break;
                    }
                nsr.myTvQ.indexedDB.ResetVersion();
            }
        };
         
        //set version back to old version
        nsr.myTvQ.indexedDB.ResetVersion = function () {
            var db = nsr.myTvQ.indexedDB.db;
            var setVrequest = db.setVersion(nsr.myTvQ.indexedDB.version);
            setVrequest.onfailure = nsr.myTvQ.indexedDB.onerror;
            setVrequest.onsuccess = function (e) {
                console.log('reset db.version ', db.version);
            }
        };
         
        //empty or truncate table
        nsr.myTvQ.indexedDB.ClearAll = function(storeName, callback){
            var db = nsr.myTvQ.indexedDB.db;
            if (db.objectStoreNames.contains(storeName)) {
                var clearTransaction = db.transaction([storeName], IDBTransaction.READ_WRITE);
                var clearRequest = clearTransaction.objectStore(storeName).clear();
                clearRequest.onsuccess = function(event){
                     //event.target.result contains undefined
                     if (callback) {
                        callback(true);
                     }                 
                };
                clearRequest.onerror = function(event){            
                     console.log("myError: ", event);
                     if (callback) {
                        callback(false);
                     }                                  
                };
            }else {
                console.log(storeName+ " doesnot exist ");
                if (callback) {
                        callback(false);
                }                 
            }
        }
        //nsr.myTvQ.indexedDB.ListAll('','store_id',function(l1){console.log(l1);});
        nsr.myTvQ.indexedDB.ListAll = function(storeName, objOrArray, callback){
            var db = nsr.myTvQ.indexedDB.db;
            if (db.objectStoreNames.contains(storeName)) {
                var transaction = db.transaction(storeName, IDBTransaction.READ_ONLY);
                var list = objOrArray ? {} : [] ;
                transaction.oncomplete = function(event) {
                    if(callback){
                        //console.log(results);
                        callback(list);
                    }
                };
                transaction.onerror = nsr.myTvQ.indexedDB.onerror;
                var cursorRequest = transaction.objectStore(storeName).openCursor();
                 
                cursorRequest.onsuccess = function(ev) {
                    var cursor = cursorRequest.result || ev.target.result;
                    if(!!cursor == false)
                        return;
                    if(objOrArray){
                        list[cursor.value[objOrArray]] = cursor.value;
                    }
                    else{
                        list.push(cursor.value);                
                    }
                    cursor.continue();
                };      
            }else {
                console.log(storeName+ " doesnot exist ");
                if (callback) {
                        callback(false);
                }                 
            }            
        }
         
        nsr.myTvQ.indexedDB.Delete = function(storeName, key, callback){
            var db = nsr.myTvQ.indexedDB.db;
            var request = db.transaction(storeName, IDBTransaction.READ_WRITE)  
                    .objectStore(storeName)  
                    .delete(key);  
            request.onsuccess = function(event) {  
              if(callback){
                    //console.log(results);
                    callback(true);
                }  
            };
            request.onerror = function(event) {  
              if(callback){
                    callback(false);
                }  
            };
        }
         
        nsr.myTvQ.indexedDB.CreateSettingsStore = function(callback){
            var db = nsr.myTvQ.indexedDB.db;
            if(!db.objectStoreNames.contains("settings")) {
                //will be used by background to get all past dates and run operations on them
                db.createObjectStore("settings",{keyPath:"Name"});
                console.log('objectstore settings created');
                             
                nsr.myTvQ.indexedDB.SetAllSettings(undefined,callback);
            }
        };
         
         
         
        nsr.myTvQ.indexedDB.GetAllSettings = function(callback){
            var db = nsr.myTvQ.indexedDB.db;        
            var settings = {};
            var transaction = db.transaction(["settings"], IDBTransaction.READ_ONLY);
            transaction.oncomplete = function(event) {  
              if(callback){
                callback(settings);
              }
            }; 
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("settings");
            var cursorRequest = store.openCursor(IDBKeyRange.lowerBound(0));
            cursorRequest.onsuccess = function(ev) {
                var cursor = cursorRequest.result || ev.target.result;
                if(!!cursor == false)
                    return;
                //console.log(cursor.value);
                settings[cursor.value.Name] = cursor.value.Value;
                cursor.continue();
            };          
        }
     
        nsr.myTvQ.indexedDB.SetAllSettings = function(settings, callback){
             var db = nsr.myTvQ.indexedDB.db; 
             var transaction = db.transaction(["settings"], IDBTransaction.READ_WRITE);
             var store = transaction.objectStore("settings");
             transaction.oncomplete = function(event) {  
                if(callback){
                    console.log('SetAllSettings Initialized');
                    callback();
                }
             }; 
             transaction.onerror = nsr.myTvQ.indexedDB.onerror;
             settings = settings || {};
             var init_list = [{"Name":"update_time","Value":settings.update_time || (new Date()).getTime() }                          
                                ,{"Name":"enable_notification","Value":settings.enable_notification || 1}
                                ,{"Name":"default_country","Value":settings.default_country || 'US'}];
     
             $.each(init_list, function (index0, item0) {
                    var request = store.put(item0);
                    request.onsuccess = function(event) {};
                    request.onerror = function(event) {
                        console.log(event);
                    }; 
             });
        }
     
        nsr.myTvQ.indexedDB.GetSettingsFor = function(name, callback){
            var db = nsr.myTvQ.indexedDB.db;
            var resultset = null;
            var transaction = db.transaction(["settings"], IDBTransaction.READ_ONLY);
            transaction.oncomplete = function(event) {  
              if(callback){
                callback(resultset);
              }
            }; 
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("settings");
            var request = store.get(name);  
            request.onerror = function(event) {  
              console.log(event);
            };  
            request.onsuccess = function(event) {  
              resultset = request.result ? request.result.Value : undefined ;  
            };
        }
     
        nsr.myTvQ.indexedDB.SetSettingsFor = function(name, value, callback){
            var db = nsr.myTvQ.indexedDB.db;
            var transaction = db.transaction(["settings"], IDBTransaction.READ_WRITE);
            transaction.oncomplete = function(event) {  
              if(callback){
                callback(true);
              }
            }; 
            transaction.onerror = function(event) {  
              if(callback){
                callback(false);
              }
            }; 
            var store = transaction.objectStore("settings");
            var request = store.put({"Name":name,"Value":value});  
            request.onerror = function(event) {  
              console.log(event);
            };  
            request.onsuccess = function(event) {};
        }    
         
        ///Show Listings
        nsr.myTvQ.indexedDB.listing.CreateListingsStore = function(callback){
            var db = nsr.myTvQ.indexedDB.db;
            if(!db.objectStoreNames.contains("listings")) {
                var store = db.createObjectStore("listings", { keyPath: "show_id" });
                store.createIndex("countryIndex", "country", { unique: false });
                store.createIndex("country_statusIndex", "country_status", { unique: false });
                store.createIndex("show_name_Index", "show_name", { unique: false });
                console.log('objectstore listing created');  
                if(callback){
                    callback(true);
                }          
            }else {
                if(callback){
                    console.log('CreateListingsStore already exits');
                    callback(true);
                }
            }
        };
         
        nsr.myTvQ.indexedDB.listing.AddListings = function (x2j_list_new, index, callback) {
            //no need to clear, since its add or update.        
            var db = nsr.myTvQ.indexedDB.db;
            var transaction = db.transaction(["listings"], IDBTransaction.READ_WRITE);
            var count = 0;
            transaction.oncomplete = function (event) {
                if (callback) {
                    console.log('x2jShowListing Added ' + count + '/' + x2j_list_new.length);
                    callback([index, count, x2j_list_new.length]);
                }
            };
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
            console.log('AddListings looping'+index);
            $.each(x2j_list_new, function (index0, item0) {
                var request = store.put(item0);
                request.onsuccess = function (event) {
                    count++;
                    // event.target.result == show_id  
                };
            });        
        };
     
        nsr.myTvQ.listing.AddListingsInChunks = function (progressElemId, callback) {
            $(progressElemId).html('(Listing is older than 14 days, fetching latest from server)');
            nsr.myTvQ.listing.GetListingsFromTvRage(function (x2j_list_new) {
                if (!x2j_list_new) {
                    if (callback) {
                        $(progressElemId).html('(Encountered Error while fetching, please try again later.)');
                        callback(false);
                        return;
                    }
                }
                console.log('x2j_list_new now calling AddListings');
                 
                //http://stackoverflow.com/questions/8495687/split-array-into-chunks      
                var i = 0, j = x2j_list_new.length, temp_array, chunk = 500;
                $(progressElemId).html('(Got a total of '+j+' Shows)');
                //http://www.kryogenix.org/days/2009/07/03/not-blocking-the-ui-in-tight-javascript-loops
                function doWork() {
                    temp_array = x2j_list_new.slice(i, i + chunk);
                    nsr.myTvQ.indexedDB.listing.AddListings(temp_array, i, function (cnt) {
                        if (cnt[0] >= j - chunk) {
                            console.log('AddListings ALL DONE', cnt);
                            if (callback) {
                                $(progressElemId).html('(Saving Shows locally 100% Comlepete. Now generating screen. Almost there... )');
                                callback(true);
                                return;
                            }
                        }
                        $(progressElemId).html('(Saving '+j+' Shows locally, '+(((i + chunk >=j ? j: i + chunk)/j)*100).toFixed(2) +'% Done.)');
                        console.log(cnt);
                    });
                    i += chunk;
                    if (i < j) {
                        setTimeout(doWork, 100);
                    }
                };
                setTimeout(doWork, 100);            
            });
        } 
         
        nsr.myTvQ.indexedDB.listing.GetAllCountries = function (callback) {
            var db = nsr.myTvQ.indexedDB.db;
            var resultset = [];
            var transaction = db.transaction(["listings"], IDBTransaction.READ_ONLY);
            transaction.oncomplete = function(event) {  
              if(callback){
                callback(resultset);
              }
            }; 
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
     
            var countryIndex = store.index("countryIndex");
             
            var cursorRequest = countryIndex.openKeyCursor(null, IDBCursor.NEXT_NO_DUPLICATE);
            cursorRequest.onsuccess = function(ev) {
              var cursor = cursorRequest.result;
              if (cursor) {
                resultset.push(cursor.key);
                //console.log(cursor.value);//undefined
                cursor.continue();
              }
            };
        };    
     
        nsr.myTvQ.indexedDB.listing.GetListingByCountry = function (country, callback) {
            console.log("GetListingBy country");
            var resultset = [];
            var db = nsr.myTvQ.indexedDB.db;
            var transaction = db.transaction(["listings"], IDBTransaction.READ_ONLY);
            transaction.oncomplete = function(event) {  
                if(callback){
                callback(resultset);
                }
            }; 
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
     
            var countryIndex = store.index("countryIndex");
                 
            var cursorRequest = countryIndex.openCursor(new IDBKeyRange.only(country));
            cursorRequest.onsuccess = function(ev) {
                var cursor = cursorRequest.result;
                if (cursor) {
                resultset.push(cursor.value);
                     
                cursor.continue();
                }
            };
        };
     
        nsr.myTvQ.indexedDB.listing.GetListingByCountryStatus = function (country, status, callback) {
            var resultset = [];
            var db = nsr.myTvQ.indexedDB.db;
            var transaction = db.transaction(["listings"], IDBTransaction.READ_ONLY);
            transaction.oncomplete = function(event) {  
              if(callback){
                callback(resultset);
              }
            }; 
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
     
            var country_statusIndex = store.index("country_statusIndex");
            var cursorRequest = country_statusIndex.openCursor(new IDBKeyRange.only(country +'_'+ status));
            cursorRequest.onsuccess = function(ev) {
              var cursor = cursorRequest.result;
              if (cursor) {
                resultset.push(cursor.value);
                //console.log(cursor.value);
                cursor.continue();
              }          
            };
        };
         
        //from ="A", to="B"
        nsr.myTvQ.indexedDB.listing.GetCountryByRange = function (from, to) {
            var db = nsr.myTvQ.indexedDB.db;
            var resultset = [];
            var transaction = db.transaction(["listings"], IDBTransaction.READ_ONLY);
            transaction.oncomplete = function(event) {  
                console.log(resultset);
            }; 
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
     
            var countryIndex = store.index("countryIndex");
             
            var cursorRequest = countryIndex.openKeyCursor(IDBKeyRange.bound(from,to,false,true), IDBCursor.NEXT_NO_DUPLICATE);
            cursorRequest.onsuccess = function(ev) {
              var cursor = cursorRequest.result;
              if (cursor) {
                resultset.push(cursor.key);
                //console.log(cursor.value);//undefined
                cursor.continue();
              }
            };
        };
         
        //chrome doesnot support count yet.
        nsr.myTvQ.indexedDB.listing.CountListing = function (callback) {        
            var db = nsr.myTvQ.indexedDB.db;
            var transaction = db.transaction(["listings"], IDBTransaction.READ_ONLY);
            var count = 0;
            transaction.oncomplete = function(event) {  
              if(callback){
                callback(count);
              }
            };
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
     
            var keyRange = IDBKeyRange.lowerBound(0);
            var showidRequest  = store.openCursor(keyRange);
            showidRequest.onsuccess = function(ev) {
              var result = ev.target.result;
              if(!!result == false)
                return;
                           
                count++;
                result.continue();          
            };
        }
     
        nsr.myTvQ.indexedDB.listing.CountListingBy = function (country, callback) {        
            var db = nsr.myTvQ.indexedDB.db;
            var transaction = db.transaction(["listings"], IDBTransaction.READ_ONLY);
            var count=0;
            transaction.oncomplete = function(event) {  
              if(callback){
                //console.log(count);
                callback(count);
              }
            };
            transaction.onerror = nsr.myTvQ.indexedDB.onerror;
            var store = transaction.objectStore("listings");
     
            var countryIndex = store.index("countryIndex");
             
            var cursorRequest = countryIndex.openCursor(new IDBKeyRange.only(country));
            cursorRequest.onsuccess = function(ev) {
              var cursor = cursorRequest.result;
              if(!!cursor == false)
                return;
     
              count++;
              cursor.continue();
            };        
        }                           
     
    } (window.nsr = window.nsr || {}, jQuery));
