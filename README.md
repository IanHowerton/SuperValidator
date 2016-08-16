Simply drag and drop the file into your project.  It is built with extensions so there is no need to subclass uitextfield.

```
//built in validation types
case None = -1
case Zip = 0
case StreetAddress = 1
case Phone = 2
case Email = 3
case IPAddress = 4
case MACAddress = 5
case GPSCoordinate = 6
case GPSPoint = 7
case URL = 8
case CreditCard = 9
case Money = 10
case Letters = 11
case LettersWithSpaces = 12
case AlphaNumeric = 13
case AlphaNumericWithSpaces = 14
case PositiveNumbers = 15
case NegativeNumbers = 16
case WholeNumbers = 17
case PositiveFloats = 18
case NegativeFloats = 19
case Floats = 20
case Text = 21
```
Text field filters
```
let UpperCaseLetters = "ABCDEFGHIJKLKMNOPQRSTUVWXYZ"
let LowerCaseLetters = "abcdefghijklmnopqrstuvwxyz"
let AllLetters = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLKMNOPQRSTUVWXYZ"
let UpperCaseHex = "0123456789ABCDEF"
let LowerCaseHex = "0123456789abcdef"
let AllHex = "0123456789abcdefABCDEF"
let PositiveWholeNumbers = "0123456789"
let WholeNumbers = "-0123456789"
let PositiveFloats = "0123456789."
let Floats = "-0123456789."
let Email = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLKMNOPQRSTUVWXYZ0123456789_-+@.%"
let Street = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLKMNOPQRSTUVWXYZ0123456789 -#.&"
let IPAddress = "0123456789."
let Money = "0123456789.$"
let Phone = "0123456789.()- "
let Zip = "0123456789-“
```

Alternatively you can add your own using "shouldAllow(String...)"
```
func textField(textField: UITextField, shouldChangeCharactersInRange range: NSRange, replacementString string: String) -> Bool 
{
  return string.shouldAllow(Numbers, “@#$”)
}
```

Declare your own — simple expression...
```
let zip = ValidationExpression(expression: "^\\d{5}(-\\d{4})?$", description: "Zip Code",failureDescription: "Invalid Zip Code”) 
```
…and apply it to a textBox programmatically 
```
txtZip.validationExpression = zip
```
//test validation before form submission
```
txtZip.text = "72034"        
let result = txtMUID.validate() //returns ValidationResult
```
**show validation result class**
```
if result.isValid
{
  print(result.transformedString)
}
else
{
  print(result.failureMessage)
}
```
More advanced validation — adds sub-rules for more user friendly hints on how to fix a problem and text transformation / cleaning
Transformation is done BEFORE validation and is optional
```
let zip = ValidationExpression(expression: "^\\d{5}(-\\d{4})?$", description: "Zip Code",failureDescription: "Invalid Zip Code", hints: [
  ValidationRule(priority: 1, expression: "\\d{5}", failureDescription: "Zip code must be 5 characters"),
  ValidationRule(priority: 0, expression: "[0-9]+", failureDescription: "Not numbers"),
  ],interfaceBuilderAliases: ["zip","zip code"], transformText: 
    { (zipcode) in
      var myString = zipcode
      myString = myString?.stringByReplacingOccurrencesOfString(" ", withString: "")
      return myString!
    }
  ,furtherValidation:nil)
```
**Special note:**

interfaceBuilderAliases is used in the Validation class and is only relevant to the built in classes (classes referenced in the enum).  Feel free to edit the file to add your own.  The aliases are what someone can enter into IB on a textbook in order to easily apply validation.

Most advanced - hints, input text transformation, and further validation
4242 4242 4242 4243 is a credit card number that passes the regular expression check.  However, the further validation performs a Luhn check that all credit cards must pass (you can read more here: (https://en.wikipedia.org/wiki/Luhn_algorithm).
The furtherValidation closure has the transformed text as a parameter and returns a Validation Result object


**show ValidationResult object**
```
let creditCard = ValidationExpression(expression: "^(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|6(?:011|5[0-9][0-9])[0-9]{12}|3[47][0-9]{13}|3(?:0[0-5]|[68][0-9])[0-9]{11}|(?:2131|1800|35\\d{3})\\d{11})$",
            description: "Debit or Credit Card",
            failureDescription: "Invalid card",
            hints: [ValidationRule(priority: 0, expression: "\\d+", failureDescription: "Missing Numbers")],
            interfaceBuilderAliases: ["card","credit card","debit card","cc"],
            transformText:
            { (card) in
              var myString = card
              myString = myString?.condensedWhitespace.stringByReplacingOccurrencesOfString(" ", withString: “") //condensedWhitespace is a helper variable that gets rid of multiple side-by-side spaces
              return myString!
            },
            furtherValidation:{[weak self] (card) in
            if (self?.luhnTest(card))!
            {
              return ValidationResult(isValid: true, failureMessage: nil, transformedString: card)
            }
            else
            {
              return ValidationResult(isValid: false, failureMessage: "Card failed Luhn check", transformedString: card)
            }
})

func luhnTest(number: String) -> Bool
{
  let noSpaceNum = number.condensedWhitespace
  let reversedInts = noSpaceNum.characters.reverse().map
  {
    Int(String($0))
  }
  return reversedInts.enumerate().reduce(0, combine: {(sum, val) in let odd = val.index % 2 == 1
  return sum + (odd ? (val.element! == 9 ? 9 : (val.element! * 2) % 9) : val.element!)
}) % 10 == 0
}
```
