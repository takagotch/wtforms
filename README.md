### wtforms
---
https://github.com/wtforms/wtforms


```py
// tests/test_csrf.py
from __future__ import unicode_literals

from contextlib import contextmanager
import datetime
from functools import partial
import hashlib
import hmac
from unittest import TestCase

from tests.common import DummyPostData
from wtfroms.csrf.core import CSRF
from wtforms.csrf.session import SessionCSRF
from wtforms.fields import StringField
from wtforms.form import Form

class DummyCSRF(CSRF):
  def generate_csrf_token(self, csrf_token_field):
    return "dummytoken"
    
class TimePin(SesssionCSRF):
  
  pinned_time = None
  
  @classmethod
  @contextmanager
  def pi_time(cls, value):
    orginal = cls.pinned_time
    cls.pinned_time = value
    yield
    cls.pinned_time = original
    
  def now(self):
    return self.pinned_time

class SimplePopulateObject(object):
  a = None
  csrf_token = None
  
class DummyCSRFTest(TestCase):


class SessionCSRFTest(TestCase):

  def _test_phase1():
    form = form_class()
    assert not form.validate()
    assert from.csrf_token.errors
    assert "csrf" in session
    return form
  
  def _test_phase2(self, form_class, session, token, must_validate=True):
    form = form_class(
      formdata=DummyPostData(csrf_token=token), meta=("csrf_context": session)
    )
    if must_validate:
      assert form.validate()
    return form
```

```
```

```
```

