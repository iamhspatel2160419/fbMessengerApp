# fbMessengerApp
Coredata Example, chat app layout in code, collectionview in code
(1) Core data create entity. and update, delete and create them 

      func setupData() {

          clearData()

          let delegate = UIApplication.sharedApplication().delegate as? AppDelegate

          if let context = delegate?.managedObjectContext {

              let mark = NSEntityDescription.insertNewObjectForEntityForName("Friend", inManagedObjectContext: context) as! Friend
              mark.name = "Mark Zuckerberg"
              mark.profileImageName = "zuckprofile"

              let message = NSEntityDescription.insertNewObjectForEntityForName("Message", inManagedObjectContext: context) as! Message
              message.friend = mark
              message.text = "Hello, my name is Mark. Nice to meet you..."
              message.date = NSDate()

              createSteveMessagesWithContext(context)

              let donald = NSEntityDescription.insertNewObjectForEntityForName("Friend", inManagedObjectContext: context) as! Friend
              donald.name = "Donald Trump"
              donald.profileImageName = "donald_trump_profile"

              FriendsController.createMessageWithText("You're fired", friend: donald, minutesAgo: 5, context: context)

              let gandhi = NSEntityDescription.insertNewObjectForEntityForName("Friend", inManagedObjectContext: context) as! Friend
              gandhi.name = "Mahatma Gandhi"
              gandhi.profileImageName = "gandhi"

              FriendsController.createMessageWithText("Love, Peace, and Joy", friend: gandhi, minutesAgo: 60 * 24, context: context)

              let hillary = NSEntityDescription.insertNewObjectForEntityForName("Friend", inManagedObjectContext: context) as! Friend
              hillary.name = "Hillary Clinton"
              hillary.profileImageName = "hillary_profile"

              FriendsController.createMessageWithText("Please vote for me, you did for Billy!", friend: hillary, minutesAgo: 8 * 60 * 24, context: context)


              do {
                  try(context.save())
              } catch let err {
                  print(err)
              }
          }

          loadData()

      }

(2)  load core data using NSFetchRequest and execute it using its context "context.executeFetchRequest"
    
        func loadData() {
        let delegate = UIApplication.sharedApplication().delegate as? AppDelegate
        
        if let context = delegate?.managedObjectContext {
            
            if let friends = fetchFriends() {
                
                messages = [Message]()
                
                for friend in friends {
                    
                    let fetchRequest = NSFetchRequest(entityName: "Message")
                    fetchRequest.sortDescriptors = [NSSortDescriptor(key: "date", ascending: false)]
                    fetchRequest.predicate = NSPredicate(format: "friend.name = %@", friend.name!)
                    fetchRequest.fetchLimit = 1
                    
                    do {
                        
                        let fetchedMessages = try(context.executeFetchRequest(fetchRequest)) as? [Message]
                        messages?.appendContentsOf(fetchedMessages!)
                        
                    } catch let err {
                        print(err)
                    }
                }
                
                messages = messages?.sort({$0.date!.compare($1.date!) == .OrderedDescending})
                
              }
           }
        }
        
(3) two entities taken in core data model

       One is friend and second is Messages 
       There is one to many relationship Betn to them friend to many messages
       Core data property automatically created by xcode 
       
       
(4) NSFetchedResultsController is used when tableview or collectionview fetch all model data and its automatically update core data model using its methods
         
         lazy var fetchedResultsControler: NSFetchedResultsController = {
          let fetchRequest = NSFetchRequest(entityName: "Message")
          fetchRequest.sortDescriptors = [NSSortDescriptor(key: "date", ascending: true)]
          fetchRequest.predicate = NSPredicate(format: "friend.name = %@", self.friend!.name!)
          let delegate = UIApplication.sharedApplication().delegate as! AppDelegate
          let context = delegate.managedObjectContext
          let frc = NSFetchedResultsController(fetchRequest: fetchRequest, managedObjectContext: context, sectionNameKeyPath: nil, cacheName: nil)
          frc.delegate = self
          return frc
          }()
     
     
     var blockOperations = [NSBlockOperation]()
    
    func controller(controller: NSFetchedResultsController, didChangeObject anObject: AnyObject, atIndexPath indexPath: NSIndexPath?, forChangeType type: NSFetchedResultsChangeType, newIndexPath: NSIndexPath?) {
    
      // when coredata new object will be inserted this method will be called NSBlockopertaion array will append its logic 
        if type == .Insert {
            blockOperations.append(NSBlockOperation(block: { 
                self.collectionView?.insertItemsAtIndexPaths([newIndexPath!])
            }))
        }
    }
    
    
    func controllerDidChangeContent(controller: NSFetchedResultsController) {
        collectionView?.performBatchUpdates({ 
            
            for operation in self.blockOperations {
                operation.start()
            }
            
            }, completion: { (completed) in
                
                let lastItem = self.fetchedResultsControler.sections![0].numberOfObjects - 1
                let indexPath = NSIndexPath(forItem: lastItem, inSection: 0)
                self.collectionView?.scrollToItemAtIndexPath(indexPath, atScrollPosition: .Bottom, animated: true)
                
        })
      }
          
          
          
          
    
    
    
    
       
       
      
        
        



   
   
      
      
      
