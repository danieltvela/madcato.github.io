---
layout:     post
title:      "MVVM architecture pattern in iOS (improved)"
date:       2016-08-8 12:02:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   mvvm-ios-architecture-pattern
---

Inspired by [this](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52#.8t6ex2n88) post, I tried to develop an app using the MVVM architecture pattern.

This architecture pattern presents three different parts:

- A View
- A Model 
- A ViewModel

The view is the responsible of interact with the UI. In iOS is de *UIView* and *UIViewController* classes. The **Model** represent the data to be managed. An the **ViewModel** becomes an interactor between **Model** and **View**. But the special thing is how thos interactor is managed by the **View**. 

The **ViewModel** must declare a variable for each data that must be showed in the **View**. Then the **View** must declare a *observer* of the variable. This *observer* is invoked everytime the variable change, and then the **View** must present the new value to the user. Like the following code example:

{% highlight swift %}
class GreetingViewModel : ViewModel {
    let person: Person
    var greeting: String? {
        didSet {
            self.greetingDidChange?(self)
        }
    }
    var greetingDidChange: ((GreetingViewModelProtocol) -> ())?
    required init(person: Person) {
        self.person = person
    }
    func showGreeting() {
        self.greeting = "Hello" + " " + self.person.firstName + " " + self.person.lastName
    }
}

class GreetingViewController : UIViewController {
    var viewModel: GreetingViewModelProtocol! {
        didSet {
            self.viewModel.greetingDidChange = { [unowned self] viewModel in
                self.greetingLabel.text = viewModel.greeting
            }
        }
    }
}
{% endhighlight %}

Declaring a **ViewModel** this way we must design a concrete class for each **View**. This class must declare every data that the **View** requires. The **ViewModel** can preprocess every data before present it: like translate text, compound mixed strings(name, surname), change meassurament units, etc.

**ViewModel** can be also responsible for controlling navigation of the **Views**. 

The benefits of this design are:

- Separate responsabilities. **View** can be the only responsible of managing the UI.
- Testability. Separating the **Model** from the **View** using a third class is the best way to test any of both without the other.

I tried this achitecture in an app, and I really liked it.

## Improvement

Instead of creating one *didChangeXXXXXX* method for every property in the **ViewModel**, create one method didChange as an map of mehtods and use an *enum* as the index of the map.

{% highlight swift %}
enum GreetingViewField {
    case greeting
    case thing
    case price
}

protocol GreetingViewModelProtocol: class {
    var greeting: String? { get }
    var thing: String? { get }
    var price: String? { get }
    var didChange: [GreetingViewField:((GreetingViewModelProtocol) -> ())] { get set }
    init(person: Person)
    func show(field: GreetingViewField)
}

class GreetingViewModel: GreetingViewModelProtocol {
    let person: Person
    var greeting: String? {
        didSet {
            self.didChange[GreetingViewField.greeting]?(self)
        }
    }
    var thing: String? {
        didSet {
            self.didChange[GreetingViewField.thing]?(self)
        }
    }
    var price: String? {
        didSet {
            self.didChange[GreetingViewField.price]?(self)
        }
    }
    var didChange: [GreetingViewField:((GreetingViewModelProtocol) -> ())] = [:]
    required init(person: Person) {
        self.person = person
    }
    func show(field: GreetingViewField) {
        switch field {
        case .greeting:
            self.greeting = "Hello" + " " + self.person.firstName + " " + self.person.lastName
        case .thing:
            self.thing = "Bye" + " " + self.person.firstName + " " + self.person.lastName
        case .price:
            self.price = "3.8"
        }
        
    }
}


class GreetingViewController: UIViewController {
    @IBOutlet weak var greetingLabel: UILabel!
    @IBOutlet weak var showGreetingButton: UIButton!
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        self.showGreetingButton.addTarget(self, action: #selector(GreetingViewController.showGreeting), forControlEvents: .TouchUpInside)
    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
    var viewModel: GreetingViewModelProtocol! {
        didSet {
            self.viewModel.didChange[.greeting] = { [unowned self] viewModel in
                self.greetingLabel.text = viewModel.greeting
            }
        }
    }
    @objc func showGreeting() {
        viewModel.show(.greeting)
    }
}

{% endhighlight %}