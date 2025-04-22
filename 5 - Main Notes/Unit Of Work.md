Source: https://martinfowler.com/eaaCatalog/unitOfWork.html

A Unit of Work keeps track of everything you do during a business transaction that can affect the database. When you're done, it figures out everything that needs to be done to alter the database as a result of your work.