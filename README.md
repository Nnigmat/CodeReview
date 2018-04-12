# Code Rewiev of Booking System
Python-Django implemetion of Library Management System website: 
Introduction to Programming project by students of BS1-7 group, team: Danil Ginzburg, Nikita Nigmatulin,
Maria Charikova, Roman Bogachev

## Book requests

Patron cannot check out book himself, but he can leave a request for a book from library. The librarian can approve or decline request.
Request is an object that is created in the moment "request" button is pressed and is deleted when the librarian approve\decline it.
     
    class Request(models.Model):
        doc = models.ForeignKey(Document, null=True, default=None, on_delete=models.CASCADE)
        checked_up_by_whom = models.ForeignKey(User, null=True, default=None, on_delete=models.CASCADE)
        timestamp = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return str(self.doc)
       
The model itself includes ForeignKey to the document which is requested, ForeignKey to the user who made the request and a timestamp when
the request was created. 

A request cannot be created if the user already has a request for this particular document, this document is reference or user already
has this document in use. 

## Return system
    @required_staff
    def return_doc(request):
        """
        taking document back (user has returned his document)
        """
        try:
            copy_instance = documents_models.DocumentCopy.objects.get(pk=request.GET.get('copy_id'))
        except:
            return redirect('/')
        user_id = copy_instance.checked_up_by_whom.id
        copy_instance.doc.copies += 1
        copy_instance.doc.save()
        copy_instance.delete()
        return redirect('/user?id=' + str(user_id))

Only librarian can approve that a book has been returned, thus only librarian can see "return book" button. Every time this button
is pressed the document copy object is deleted and the number of available copies of the document is increased by 1. 

## Renew
     @need_logged_in
     def renew(request):
         """
         updates returning date of document for one additional week
         (if days left less then 1, no outstanding requests and it was not renewed before)
         """
         user = request.user
         copy = None
         try:
             copy = user.documentcopy_set.get(id=request.GET.get('copy_id'))
         except:
             return HttpResponse('forbidden')
         returning_date = user.documentcopy_set.get(doc=copy.doc).returning_date
         returning_date = datetime.datetime(year=returning_date.year,
                                            month=returning_date.month,
                                            day=returning_date.day,
                                            hour=returning_date.hour)
         time_left = returning_date - datetime.datetime.today()
         days_for_checking_out = 21  # for student
         if user.userprofile.status == 'visiting professor':
             days_for_checking_out = 7
         elif copy.doc.type == "AVFile" or copy.doc.type == "JournalArticle":
             days_for_checking_out = 14
         elif user.userprofile.status in ['instructor', 'TA', 'professor']:
             days_for_checking_out = 28
         elif copy.doc.is_bestseller:
             days_for_checking_out = 14
             
Function renew gets request from user who have document and want to increase the time until he need to return it back. Each type of users have its own time in which he can renew the document. Also user can only once make renew except when document is outstanding request and no one user can renew it.
             
