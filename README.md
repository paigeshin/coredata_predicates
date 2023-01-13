# coredate_predicates


# Predicate

[https://kapeli.com/cheat_sheets/NSPredicate.docset/Contents/Resources/Documents/index](https://kapeli.com/cheat_sheets/NSPredicate.docset/Contents/Resources/Documents/index)

%K => #keyPath

%@ => String, Date

%i => Number

BEGINSWITH[cd] ⇒ case insensitive, dialect 

```swift
import Foundation
import CoreData

extension Review: BaseModel {
    
    static func getReviewsByMovieId(movieId: NSManagedObjectID) -> [Review] {
        
        let request: NSFetchRequest<Review> = Review.fetchRequest()
        request.predicate = NSPredicate(format: "movie = %@", movieId)
        
        do {
            return try viewContext.fetch(request)
        } catch {
            return [] 
        }
        
    }
    
}
```

```swift
import Foundation
import CoreData

extension Movie: BaseModel {
   
    static func byMinimumReviewCount(minimumReviewCount: Int = 1) -> [Movie] {
        
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        //request.predicate = NSPredicate(format: "reviews.@count >= %i", minimumReviewCount)
        request.predicate = NSPredicate(format: "%K.@count >= %i", #keyPath(Movie.reviews) ,minimumReviewCount)
        
        do {
            return try viewContext.fetch(request)
        } catch {
            return []
        }
        
    }
    
    static func byMovieTitle(title: String) -> [Movie] {
        
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.predicate = NSPredicate(format: "%K BEGINSWITH[cd] %@", #keyPath(Movie.title), title)
        
        do {
            return try viewContext.fetch(request)
        } catch {
            return []
        }
    }
    
    
    static func byDateRangeOrMinimumRating(lower: Date?, upper: Date?, minimumRating: Int?) -> [Movie] {
        
        var predicates: [NSPredicate] = []
        
        if let lower = lower, let upper = upper {
            
            let dateRangePredicate = NSPredicate(format: "%K >= %@ AND %K <= %@",
                                                            #keyPath(Movie.releaseDate),
                                                            lower as NSDate,
                                                            #keyPath(Movie.releaseDate),
                                                            upper as NSDate)
            
            predicates.append(dateRangePredicate)
            
        } else if let minRating = minimumRating {
            
            let minRatingPredicate = NSPredicate(format: "%K >= %i", #keyPath(Movie.rating), minRating)
            predicates.append(minRatingPredicate)
        }
        
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.predicate = NSCompoundPredicate(orPredicateWithSubpredicates: predicates)
        
        do {
            return try viewContext.fetch(request)
        } catch {
            return []
        }
        
    }
    
    static func byDateRange(lower: Date, upper: Date) -> [Movie] {
        
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.predicate = NSPredicate(format: "%K >= %@ AND %K <= %@",
                                        #keyPath(Movie.releaseDate),
                                        lower as NSDate,
                                        #keyPath(Movie.releaseDate),
                                        upper as NSDate
        )
        
        do {
            return try viewContext.fetch(request)
        } catch {
            return []
        }
        
    }
    
    
    static func byReleaseDate(releaseDate: Date) -> [Movie] {
        
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.predicate = NSPredicate(format: "%K >= %@",#keyPath(Movie.releaseDate), releaseDate as NSDate)
        
        do {
            return try viewContext.fetch(request)
        } catch {
            return []
        }
    }
    
    
    static func byActorName(name: String) -> [Movie] {
        
        let request: NSFetchRequest<Movie> = Movie.fetchRequest()
        request.predicate = NSPredicate(format: "actors.name CONTAINS %@", name)
        //request.predicate = NSPredicate(format: "%K.name CONTAINS %@", #keyPath(Movie.actors), name)
        //request.predicate = NSPredicate(format: "%K.%K CONTAINS %@", #keyPath(Movie.actors), #keyPath(Actor.name), name)
        
        do {
            return try viewContext.fetch(request)
        } catch {
            print(error)
            return []
        }
        
    }
    
}
```

### SORT

```swift
# Sort

```swift
func sort() {
      let request: NSFetchRequest<Movie> = Movie.fetchRequest()
      
      // selectedSortOptions => title, releaseDate, rating
      // selectedSortDirection => true, false
      request.sortDescriptors = [NSSortDescriptor(key: selectedSortOption.rawValue, ascending: selectedSortDirection.value)]
      let fetchedResultController: NSFetchedResultsController<Movie> = NSFetchedResultsController(fetchRequest: request,
                                                                                                  managedObjectContext: CoreDataManager.shared.viewContext,
                                                                                                  sectionNameKeyPath: nil,
                                                                                                  cacheName: nil)
      try? fetchedResultController.performFetch()
      DispatchQueue.main.async {
          self.movies = (fetchedResultController.fetchedObjects ?? []).map(MovieViewModel.init)
      }
  }
```
```
