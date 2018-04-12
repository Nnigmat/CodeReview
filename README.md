# Code Review of Booking System
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


