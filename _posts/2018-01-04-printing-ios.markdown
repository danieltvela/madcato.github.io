---
layout:     post
title:      "Printing in iOS"
date:       2018-01-03 01:00:00
author:     "Daniel Vela"
og_image: "/img/post-bg-04.jpg"
locale:       en
lang-ref:   printing-ios
---
## Doc 

- [Apple guide](https://developer.apple.com/library/content/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/Printing/Printing.html)
- [Apple AirPrint](https://developer.apple.com/airprint/)
- [Where is printer simulator in Xcode 6](https://stackoverflow.com/questions/26030702/where-is-printer-simulator-in-xcode-6)

## Install

In order to test the printing system from the *iOS Simulator* is required to install the **'Printer Simulator'**. This app is included inside [Additional Tools for Xcode 9](https://download.developer.apple.com/Developer_Tools/Additional_Tools_for_Xcode_9/Additional_Tools_for_Xcode_9.dmg) (inside folder **Hardware**) in [Developer Downloads](https://developer.apple.com/download/)

## Usage

To print for iPhone/iPad is necesary to obtain the shared instance of **UIPrintInteractionController** y configure it. 
To do that, you must create an instance of **UIPrintInfo** an set all the necessary values as:

- Work name to identify it in the print queue
- Orientation
- Type of impression: text, image, other.
- The content to print, in one of the follogin formats:
	- NSData, NSURL, UIImage, o ALAsset 
	- Array of NSData, NSURL, UIImage, or ALAsset 
	- UIPrintFormatter (this allows to define the layout)
	- UIPrintPageRenderer 
	

Last, you must to present the controller in the screen using some method like **presentFromBarButtonItem**.

## Sample Code

{% highlight objc %}
- (IBAction)btnPrintTapped:(id)sender {
    [self doPrint];
}

- (void)doPrint {
    UIPrintInteractionController* controller = [self configurePrinting:@"printerX"];
    NSURL* url = [self urlForPDF];
    controller.printingItem = url;
    [self printWithController:controller];
}

- (UIPrintInteractionController*)configurePrinting:(NSString*)printerId {
    UIPrintInteractionController* controller = [UIPrintInteractionController sharedPrintController];
	
    UIPrintInfo* printInfo = [UIPrintInfo printInfo];
    // Configure printinfo
    printInfo.jobName = @"Sextante app print";
    printInfo.orientation = UIPrintInfoOrientationPortrait;
    printInfo.outputType = UIPrintInfoOutputGeneral; // For photos and text
//    printInfo.printerID = // Se puede indicar la anterior impresora

    controller.printInfo = printInfo;
    controller.delegate = self;
	
    return controller;
}

- (void)printWithController:(UIPrintInteractionController*)controller {
    [controller presentFromBarButtonItem:self.btnPrint
                                animated:YES
                       completionHandler:^(UIPrintInteractionController * _Nonnull printInteractionController, BOOL completed, NSError * _Nullable error) {
						   
                       }
     ];
}


# pragma mark - Printer delegate
//- ( UIViewController * _Nullable )printInteractionControllerParentViewController:(UIPrintInteractionController *)printInteractionController
//
//- (UIPrintPaper *)printInteractionController:(UIPrintInteractionController *)printInteractionController choosePaper:(NSArray<UIPrintPaper *> *)paperList;
//
//- (void)printInteractionControllerWillPresentPrinterOptions:(UIPrintInteractionController *)printInteractionController;
//- (void)printInteractionControllerDidPresentPrinterOptions:(UIPrintInteractionController *)printInteractionController;
//- (void)printInteractionControllerWillDismissPrinterOptions:(UIPrintInteractionController *)printInteractionController;
//- (void)printInteractionControllerDidDismissPrinterOptions:(UIPrintInteractionController *)printInteractionController;
//
//- (void)printInteractionControllerWillStartJob:(UIPrintInteractionController *)printInteractionController;
- (void)printInteractionControllerDidFinishJob:(UIPrintInteractionController *)printInteractionController {
	
    int a = 0 ;
}

//- (CGFloat)printInteractionController:(UIPrintInteractionController *)printInteractionController cutLengthForPaper:(UIPrintPaper *)paper NS_AVAILABLE_IOS(7_0);
//- (UIPrinterCutterBehavior) printInteractionController:(UIPrintInteractionController *)printInteractionController chooseCutterBehavior:(NSArray *)availableBehaviors NS_AVAILABLE_IOS(9_0);
{% endhighlight %}