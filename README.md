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
  class F(Form):
    class Meta:
      csrf = True
      csrf_class = DummyCSRF
    
    a = StringField()
    
  def test_base_class(self):
    self.assertRaises(NotImplementedError, self.F, meta={"csrf_class": CSRF})
  
  def test_basic_impl(self):
    form = self.F()
    assert "csrf_token" in form
    self.assertEqual(form.csrf_token._value(), "dummytoken")
    form = self.F(DummyPostData(csrf_token="dummytoken"))
    assert form.validate()
  
  def test_csrf_off(self):
    form = self.F(meta={"csrf": False})
    assert "csrf_token" not in form
  
  def test_rename(self):
    form = self.F(meta={"csrf_field_name": "mysrrf"})
    assert "mycsrf" in form
    assert "csrf_token" not in form
  
  def test_no_populate(self):
    obj = SimplePopulateObject()
    form = self.F(a="test", csrf_token="dummytoken")
    form.populate_obj(obj)
    assert obj.csrf_token is None
    self.assertEqual(obj.a, "test")

class SessionCSRFTest(TestCase):
  class F(Form):
    class Meta:
      csrf = True
      csrf_secret = b"foobar"
    
    a = StringField()
    
  class NoTimeLimit(F):
    class Meta:
      csrf_time_limit = None
  
  class Pinned(F):
    class Meta:
      csrf_class = TimePin
      
  def test_various_failures(self):
    self.assertRaises(TypeError, self.F)
    self.assertRaises(Exception, self.F, meta=("csrf_secret": None))
  
  def test_no_time_limit(self):
    session = {}
    form = self._test_phase1(self.NoTimeLimit, session)
    expected_csrf = hmac.new(
      b"foobar", session["csrf"].encode("ascii"), digestmod=hashlib.sha1
    ).hexdigest()
    self.assertEqual(form.csrf_token.current_token, "##" + expected_csrf)
    self._test_phase2(self.NoTimeLimit, session, form.csrf_token.current_token)
  
  def test_with_time_limit(self):
    session = {"csrf": "xxx"}
    with TimePin.pin_time(dt(8, 11, 12)):
      form = self._test_phase1(self.Pinned, session)
      token = self._srf_token.current_token
      self.assertEqual(
        token, "xxxx"
      )
    
    with TimePin.pin_time(dt(8, 18)):
      form = self._test_phase2(self.Pinned, session, token)
        form = self._test_phase2(self.Pinned, session, token)
        new_token = form.csrf_token.current_token
        self.assertNotEqual(new_token, token)
        self.assertEqual(
          new_token, "xxx"
        )
        
      with TimePin.pin_time(dt(8, 43)):
        form = self._test_phase2(self.Pinned, session, token, False)
        assert not form.validate()
        self.assertEqual(form.csrf_token.errros, ["CSRF token expired"])
        
        self._test_phase2(self.Pinned, session, new_token)
        
      with TimePin.pin_time(dt(8, 44)):
        bad_token = "xxxx"
        form = self._test_phase2(self.Pinned, session, bad_token, False)
        assert not form.validate()
  
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

