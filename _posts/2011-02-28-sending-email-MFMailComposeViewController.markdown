---
layout:     post
title:      "Sending mail with MFMailComposeViewController"
date:       2011-02-28 12:31:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
locale:       en
lang-ref:   sending-email-MFMailComposeViewController
---

First include **MessageUI.framework** an implement **MFMailComposeViewControllerDelegate**.

Implement a method like this one an the delegate method for removing the mail controller.

{% highlight objc %}
- (IBAction)pressentMailController:(id)sender {
 MFMailComposeViewController *picker = [[MFMailComposeViewController alloc] init];
 picker.mailComposeDelegate = self;
 [picker setSubject:@"Test subject!"];
 // Set up the recipients.
 /*NSArray *toRecipients = [NSArray arrayWithObjects:@"first@example.com", nil];
 NSArray *ccRecipients = [NSArray arrayWithObjects:@"second@example.com", @"third@example.com", nil];
 NSArray *bccRecipients = [NSArray arrayWithObjects:@"four@example.com", nil];
 [picker setToRecipients:toRecipients];
 [picker setCcRecipients:ccRecipients];
 [picker setBccRecipients:bccRecipients];
 */
 // Attach an image to the email.
 /*NSString *path = [[NSBundle mainBundle] pathForResource:@"ipodnano"
 ofType:@"png"];
 NSData *myData = [NSData dataWithContentsOfFile:path];
 [picker addAttachmentData:myData mimeType:@"image/png"
 fileName:@"ipodnano"];
 */
 // Fill out the email body text.
 NSString *emailBody = @"It is raining in sunny California!";
 [picker setMessageBody:emailBody isHTML:NO];
 // Present the mail composition interface.
 [self presentModalViewController:picker animated:YES];
 [picker release]; // Can safely release the controller now.
 }
 // The mail compose view controller delegate method
 - (void)mailComposeController:(MFMailComposeViewController *)controller
 didFinishWithResult:(MFMailComposeResult)result
 error:(NSError *)error
 {
 [self dismissModalViewControllerAnimated:YES];
 }
 
{% endhighlight %}

The code allows to add recipients, body, subject and attachements. Uncomment the lines as needed.

![mail controller]({{ site.url}}/assets/tumblr_inline_mjpfxhMYM21qz4rgp.png)