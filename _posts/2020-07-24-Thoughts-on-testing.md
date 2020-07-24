---
layout: post
title: Thoughts on Testing
tags: testing
---

One of the problems that I run into a lot when working on projects is the ability to actually test the code. I might be beating a dead horse, but testing is one of those tasks that all too often gets dropped by the wayside, and honestly, I find it fun at times.

The first part of the process (which is still ongoing, mind you) was to determine what sort of code environment I was going to be working with. I found that I was working on a [Python](https://www.python.org/) project utilizing [SQLAlchemy](https://www.sqlalchemy.org/) and [pandas](https://pandas.pydata.org/). I don't think this is necessarily a complex project in it's own right, since the majority of the project is just dealing with processing data. I wasn't too worried, since I had previously worked on getting a similar testing setup working. This simplifies things because now I really only need to prepare one aspect of the tests -- the database.

I started by using the [transaction_fixtures.py](https://gist.github.com/ProvoK/10101902b1f56be946a99d576cbb397f#file-transaction_fixtures-py) file by [Vittorio Camisa](https://medium.com/@vittorio.camisa), documented [here](https://medium.com/@vittorio.camisa/agile-database-integration-tests-with-python-sqlalchemy-and-factory-boy-6824e8fe33a1). The idea is that I have a SQLAlchemy database, and I want to make sure that I can test things in a controlled environment. I've adapted it to the following:

```python
from typing import Iterator

import pytest
from sqlalchemy import create_engine
from sqlalchemy.engine import Engine, Connection
from sqlalchemy.orm import sessionmaker

from model import Base

Sessionmaker: sessionmaker = sessionmaker()


class AbstractSqlalchemyTest(object):
    @pytest.fixture
    def session(self, connection: Connection) -> Iterator[Session]:
        """
        For every test generate a new session. Roll back all changes on the session via transaction.
        """
        transaction = connection.begin()
        session = Sessionmaker(bind=connection)
        yield session
        session.close()
        transaction.rollback()
        
	@pytest.fixture(scope='module', autouse=True)
    def set_up_database_structure(self, engine: Engine) -> None:
        """
        Add all of the tables/views/etc. into the DB before running anything.
        """
        Base.metadata.create_all(engine)
        
    @pytest.fixture(scope='module')
    def connection(self, engine: Engine) -> Iterator[Connection]:
        """
        Generate a connection to the DB. Use https://docs.pytest.org/en/5.4.3/fixture.html#fixture-finalization-executing-teardown-code
        for a simple automated open/close.
        """
        connection = engine.connect()
        yield connection
        connection.close()
        
    @pytest.fixture(scope='module')
    def engine(self) -> Engine:
        """
        Generate the Engine. Use an in-memory DB.
        """
        return create_engine('sqlite:///:memory:')
```

The main idea here is that I want to generate a "new" DB for every module, set up the connection per-module, and then set up the tables per-module. Per-test we are getting a session that will auto-rollback when the test is completed (when the fixture goes out-of-scope). This means that every test uses an ephemeral DB, and we need to load the data into the DB on a per-test basis.

If it's not obvious, we are using [pytest](https://docs.pytest.org/en/stable/) as the testing framework. The thing I enjoy most about it is that I can use fixtures on a per-test basis as needed, which means that I can write a lot of fixtures for generating mock objects and then just pull them into the test manually. Pretty nice, especially when it comes to closing resources after the test is complete.

From there, the testing becomes slightly easier. The test can take a pytest fixture and use it as the core database link for the test. A test might look like this:

```python
# SQLAlchemy Models
class MacGuffin(Base):
    pass

class MacGuffinData(Base):
    pass

def func1(session: Session):
    macguffins: List[MacGuffin] = session.query(MacGuffin).all()
	mgdata: List[MacGuffinData] = map(lambda mg: MacGuffinData(...), macguffins)
    session.add_all(mgdata)

# Tests
class TestThing(AbstractSqlalchemyTest):
	def test_thing1(self, session: Session):
        macguffins: List[MacGuffin] = [MacGuffin(...) for i in range(0, 10)]
        session.add_all(macguffins)
        
        assert 10 == len(session.query(MacGuffin).all())
        
        func1(session)
        
        assert 10 == len(session.query(MacGuffinData).all())
```

This is a simple example, but this is generally what we are looking for now. We can generate some `MacGuffin`s, add them to the temporary database, and from there we can query the database about them when needed.

**Note**: I'm writing tests for things that aren't necessarily written the best way. `func1` is written so that it takes in only the Session and from there pulls data, processes it, and stores the results to the database. This isn't my preferred way of doing things, but this is how the code was written when it was handed to me.

At this point I have functions to test, a framework to support testing it, and now I just need to write tests.

---

I think this is a good point to stop. We've currently got the framework for testing, as well as some example ways to test. Whenever I write about property-based testing, I will link it here to flesh out how I'm writing the tests and how I'm thinking about the code further.