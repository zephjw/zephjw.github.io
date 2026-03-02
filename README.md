
<!DOCTYPE html>
<html>


    <head>      

        <title>John Zephyr Portfolio</title>
    </head>         
         
    <body>  
        <b>Software Design and Engineering</b>
        <h2>The Weight Tracker: An Android Health Application</h2>
                
        <h2> Android Health Application </h2>
        <b> Description </b><br>
            <p>
               The Weight Tracker is an Android application built in Kotlin during my third year of the Computer Science program. Its purpose is to allow users to log their body weight over time, visualize trends via a                   line chart, and manage personal health data, including glucose, insulin, and sleep records. The application uses Android's Jetpack component suite, including Room for local persistence, Fragments for UI                    composition, and ViewModel for state management. The artifact selected for this category is the full application codebase, with specific focus on the Fragment layer, ViewModel, Repository pattern, and XML layout files.
                Specific enhancements I made to the Software Design and Engineering artifact:
                                
                •Replacing hardcoded dp dimensions across all layout files with `match_parent`, `wrap_content`, or `0dp` with constraints, making the UI responsive to different screen sizes and font scale settings.
                •Replacing a ListView used for gender selection in InitialUserDataFragment with a `MaterialButtonToggleGroup`, eliminating an outdated component and replacing it with the correct Material Design pattern                     for a small mutually-exclusive selection.
                •Fixing missing android: id attributes on layout components that were referenced in fragment code, which would have caused `NullPointerExceptions` at runtime.
                •Adding a Repository layer between the ViewModel and DAO, correctly implementing the separation of concerns that MVVM requires.
                •Fixing a critical bug in `InitialUserDataFragment` where `android.R.attr.height`, an Android resource attribute constant, was being saved to `SharedPreferences` instead of the user's actual height value    
                Reflection on the Enhancement Process
                The key lesson I learned was the difference between code that works and code that is correct. Many bugs in the original application didn’t show up right away, such as hardcoded dimensions that looked                       fine on one device but broke on others, or a `SavedPreferences` entry storing garbage data. This experience shifted my approach to code review, leading me to question whether code not only runs but also                    functions as intended.
                The toughest challenge was working with Android’s ConstraintLayout, which requires precise constraints. Missing a constraint can cause unexpected layout issues that only become apparent on different                        devices or accessibility settings. Methodically reviewing each layout file helped me develop a thorough testing habit.
            </p>
            
        <b>Databases</b>
             The Room entity classes, DAO interface, TypeConverters, and Repository implementation.
             The database layer was the most technically broken part of the original application and, therefore, represents the most substantive demonstration of growth. The original SQLite, then the converted DAO,                     contained methods that could not compile `@Insert` operations accepting primitive types and individual field values rather than entity objects, `@Update` methods returning types that Room does not support,                 duplicate method signatures, and query methods that filtered on the wrong column with the wrong parameter type. 
             
        <p>Specific enhancements made to the Database</p> 
             <list>
                •Redesigning the `InitialUserData` entity to use Float for weight and height (replacing Int), a proper Gender enum (replacing a plain String), a foreign key relationship to the User entity with a CASCADE                    delete rule, and an index on the userId column for query performance.
                •Creating a TypeConverter class to allow Room to persist the Gender enum as a String in the database while preserving type safety in application code.
                •Replacing the bloated DAO of over 30 methods with a correct, minimal interface containing only insert, update, getByUserId, and getAll — the only operations the application requires. I incorporated                         practices from SQLite, which ultimately increased the complexity of the Data Access Object (DAO).
                •Making all DAO methods suspend functions to enforce off-main-thread execution, and returning `Flow<`List` <`InitialUserData`>>` from the query method to enable reactive UI updates.
                •Implementing the Repository as a proper class with an @Inject constructor and suspend functions wrapping DAO calls in Dispatchers.IO, rather than the annotation class that could not compile.
              </List>
             <b>Reflection on the Enhancement Process</b>   
             <p>
                 The key lesson learned from this enhancement process was the importance of understanding the conceptual role of each layer in the database stack. The original Data Access Object (DAO) was designed as a                    service class, responsible for field-level operations, business logic, and data transformation. This misunderstanding led to the creation of methods such as `insertAge(age: Int)` and `updateGender (gender:                   String)`,       which treated the database as a collection of independent fields rather than as a set of related entities. By understanding that the DAO's primary role is to translate between Kotlin                       objects and SQL rows, we can resolve the issue of over-engineering by ensuring that all additional logic is placed within the ViewModel or Repository.
             </p>
             <b>Algorithms and Data Structures</b>
             <h2>Search, Filter, and Sort Feature</h2>
             <b>Returns all entries ordered by date ascending, used for the chart</b>
             <p>Supports partial matches: "2024" returns full year, "2024-03" returns one month</p>
                
            kotlin
                fun searchByDate(query: String) {
                    if (query.isBlank()) {
                            loadAllEntries()
                            return
                    }
                    viewModelScope.launch {
                    val results = repository.searchByDate(query)
                    _entries.value = applySortOrder(results, _sortOrder.value)
                    _isFiltered.value = true
                    }
                }
      
            This function performs a partial string match search against date values.             
            Supports incremental filtering:
            *"2024" → returns all entries from that year*
            *"2024-03" → returns entries from March 2024*
            Search Strategy:
            SQL LIKE pattern matching (linear scan in the worst case).
             
            Time Complexity:
             O(n) for filtering unless indexed.
             Optimization Consideration:
             Date fields can be indexed in SQLite to improve query performance.
             <p>Design Considerations</p>
             <ul>
                Uses `viewModelScope.launch` to execute asynchronously.
                Applies sorting after filtering to maintain consistent UI behavior.
                Maintains a `_isFiltered` flag to control UI state
             </ul> <br>
                I am working on implementing this query that retrieves all WeightEntry records sorted by date in ascending order directly at the database level.
                *And this is the plan of how it will be implemented:*
                kotlin
                @Query("SELECT * FROM weight_entries ORDER BY date ASC")
                    fun getAllEntriesAsc(): Flow<`List` `<`WeightEntry`>>
                Algorithmic Complexity:
        <div>
            Sorting is handled by SQLite’s internal sorting algorithm `O(n log n)`.
                
            Data Structure Used:
            Results are returned as a `Flow<`List`<WeightEntry>>`, which emits updates reactively when the underlying table changes.
                Use Case:
                Ensures chronological ordering for data visualization
                This approach demonstrates proper separation of concerns: persistent ordering is handled at the storage layer.
        </div>     
        <div>
            Sort
               In-memory comparator `sort — O(n log n)` via Kotlin's Timsort implementation
                 Applied after every fetch, so the sort order persists across filter changes
             kotlin
                 fun toggleSortOrder() {
                     val newOrder = if (_sortOrder.value == SortOrder.ASCENDING)
                         SortOrder.DESCENDING else SortOrder.ASCENDING
                     _sortOrder.value = newOrder
                     _entries.value = applySortOrder(_entries.value, newOrder)
                 }         
        </div>
        <br>
        <div>   
            Sorting is performed in memory using Kotlin’s built-in sorting functions.
            Algorithm Uses Kotlin’s sortedBy / sortedByDescending uses Timsort.
                
            Time Complexity:
            `O(n log n)` worst case
            O(n) best case (nearly sorted data)
                
            Highly optimized for partially sorted datasets
                                
            Stable sorting (preserves the relative order of equal elements)
                                
            Design Rationale
                                
            Sorting is applied after every fetch.                
            Sort order persists across filtering operations.
                                
            In-memory sorting avoids repeated database queries when only order changes.
        </div>     
        <b>Algorithm Type: Linear Search</b>
        <div>
            kotlin
            fun findFirstEntryByExactDate(date: String): WeightEntry? {
            for (entry in _entries.value) {
                if (entry.date == date) return entry  // early exit
                }
                return null
            }
            Time Complexity:
            Best Case: `O(1)`
            Worst Case: `O(n)`
            - Optimization:
            Early exit reduces average-case runtime.
            - Use Case:
                Quickly navigate to a specific record without querying the database again.
                Given that _entries. value is already a relatively small in-memory list, a linear search is more efficient than performing another database query. For larger datasets, a HashMap<`String`, WeightEntry>                      index could reduce lookup time to O(1).
        </div>    
    </body>
    <footer>
        <a href="aboutme.html">About ME</a>
    </footer>                
</html>
