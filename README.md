# Disguise-O-Rama Spec:

## Splash Screen

- Layout via app_image_disguise, app_image_owithscotch, app_image_rama
    - Layout in launchscreen.xib via 3 uiimageviews
    - using contraints center “app_image_owithscotch" in the view
    - center the other two images horizontally
    - align the top of “app_image_rama" to the bottom of the “owithscotch" plus 10.
    - align the bottom of “app_iamge_disguise" to the top of “app_image_rama" plus 50

## IntroViewController Class

- In initialization call createIntroTopView and createIntroBottomView
- Create createIntroTopView method (private
    - View the width of the screen and (percentage based on spec) height.
    - Place app_image_line in a uiimageview and align to bottom of view
    - add a uiscrollview to the top of the view the width of the screen and the height of the view minus the height of the bottom line image.  This scrollview should be owned by a property.
    - Font is UIFont.introFont and the text color is black
    - In that scrollview add three labels, each with numberoflines set to zero.
        - The first should be at x of zero and say “S’up player…"
        - The second should be at x of the view’s width and say “OK. Long story short… There are some government thugs out there looking for me."
        - The third should be at x of two times the view’s width and say “I need you to help me pick a costume so that I can run to the liquor store without anyone recognizing me."
        - Put these strings into an array corresponding to the button array below and layout by enumerating through array and adding labels.
- Create createIntroBottomView method (private)
    - One Button the width of the screen and (percentage based on spec) height.
    - Background color is #E15926
    - Font is UIFont.introButtonFont and the titleColor is white
    - Create array of strings called introButtonTitles containing [“Say \”hello\””, “Tell me more”, “OK”] as well as set the value of a property currentIntroPage set to zero.
    - set title of button to introButtonTitles[currentIntroPage]
    - add a target to the button that is “nextIntroPage"
- Create nextIntroPage method (private)
    - currentIntroPage++
    - If the currentIntroPage is greater than the introButtonTitles count, call introCompleted delegate method
    - Otherwise Scroll the scrollview to currentIntroPage * self.view.width animated
        - change the title on the button to introButtonTitles[currentIntroPage]
- Create delegate callback with a method introCompleted
- Create animateIn method (public)
    - Set bottom of topIntroView to zero
    - Set top of bottomIntroView to the screen height
    - In a animateWithDuration Block (linear).
        - animate in the topIntroView so the top is at zero
        - animate in the bottomIntroView so the bottom is at the height of the screen.
- Create animateOut method (public)
    - In a animateWithDuration Block (linear).
        - animate out the topIntroView so the bottom is at zero
        - animate out the bottomIntroView so the top is at the height of the screen.

## DisguiseSceneViewController Class

- class inherets from SKScene
- in initMethod populate array with fixture data from config.plist
- Add Scotch as a SpriteNode scaled to the size of the launch image
- Create updateDataMethod that is public
    - This method will update the data array that is filled with fixtureData
    - It will then call relayoutDisguises
- Create scaleScotchTo method that takes one of two values in an enum
    - Intro Size
    - Game Size
- Create animateToAngle method (takes an angle and bool use Spring
    - animate each node in the scene to the following locations using the pseudocode
        - var angle = angle passed to method
        - step = (2*PI) / disguises.count;
        - node.centerX = round(node.width/2 + circleRadius * cos(angle))
        - node.centerY = round(node.height/2 + circleRadius * sin(angle))
        - angle += step
    - if the method params specify to use a spring, animate with a spring animation
- Create animateIn method (public)
    - This method adds all the disguises via SKNode’s with their center point as the view
        - add “ring” to the bottom of the sknode and the the corresponding cached image to the top as skSpriteNodes.
        - call animateToAngle method with 0 as the angle
- Add touchesBegan, touchesMoves and touchesEnded methods as overrides
    - In touchesBegan take a point as the start location and a time as the startTime(for velocity)
    - In touchesMoved calculate the offset and move the wheel but don’t set it spinning yet
    - In touchesEnded
        - Calculate the dx and dy based on the end location minus the start location
        - If the difference is smaller than a minimum swipe distance, check the nodesAtPoint of this location for a tapGesture
            - if there is a node at this point and the current state is nothing selected, call the delegate disguiseSelected with the data for this node
                - call moveNodeToDisguise
            - Otherwise call moveBackToDisguiseSelection and the disguiseDeselected
        - If it is larger calculate the speed of the swipe using the time difference and the distance
            - call applyMotion to wheel
- Create moveNodeToDisguise method
    - set current state to disguise selected
        - move and scale the circle sprite node off the screen
        - move selected node above scotch and scale up and move onto top of him with a spring
        - apply gravity to the physics bodies of the rest of the nodes.
- Create moveBackToDisguiseSelection method
    - set current state to nothing selected
    - remove gravity from all nodes
    - call animateToAngle method with zero and spring applied
- Create applyMotion method
    - This method takes the current velocity, decays it and simply calls animateToAngle with no spring after it updates the current angle.

## DisguiseMainViewController Class (Primary ViewController class)

Setup

- In viewDidLoad call updateData (see API consumption section) and allocate the introViewController, conform this class to the introViewControllerDelegate and implement introCompleted, conform to the desguiseSceneViewControllerDelegate and implement disguiseSelected and disguiseDeselected
- Create a createDisguiseInformationView method
    - Two labels, center aligned, each connected to an outlet
    - Top disguiseTitleLabel's font is UIFont.titleFont and the color is #E15926
    - Bottom disguiseSubtitleLabel’s font is UIFont.subTitleFont and the color is black
- Add properties links to the “app_image_rama”, “app_image_o” and “app_image_disguise” laid out identically to launch screen, add all to view for transition from splash into app
- Add the disguiseSceneView to the view

Transition from screenshot (viewDidAppear):

- in a UIView animateWithDuration block
    - Move the top uiimageview off of the top of the screen
    - Move the bottom uiimageview off of the bottom of the screen
    - Scale the “app_image_o" till it is off screen
    - Call disguiseSceneView’s scaleScotchTo method with .Intro
    - on completion: remove top, bottom and "app_image_o" uiimageviews from superview call intoViewController’s  animateIn method

Transition to mainscreen:

- In the introCompleted method (conforming to introViewController’s delegate)
    - Call animateOut on introViewController
    - Call disguiseSceneView’s scaleScotchTo method with .Main
    - Call the disguiseSceneViewController animateInMethod

Disguise selection/deselection:

- Create disguiseSelected method to conform to the delegate
    - method takes dictionary object containing data
    - set disguiseTitleLabel text to xxx
    - set disguiseSubtitleLabel text to yyyy
    - set x to screen width and y to zero and add to view
    - animate in with an ease out to x = 0
- Create disguiseDeselected method to conform to the delegate
    - animate out with an ease out to right = 0
    - remove from view on completion

API Consumption

- create updateData method
- create NSURLRequest with http://ivdemo.frenchgirlsapp.com/api/current_costumes.json

- Using NSURLConnection sendAsynchronousRequest check if there is no error and the length of the data is greater than one
- If it is use NSJSONSerialization JSONObjectWithData to parse the server response into an object as a dictionary
- Download and cache each server image
- call updateData method on disguiseSceneView with this dictionary

## Miscellaneous Tasks

- Load assets into project
- Add config.plist to project for predefined costumes
- Declare UIFont+Styles category to declare font styles
    - UIFont.introFont = HelveticaNeue-Bold - 16pt
    - UIFont.introButtonFont = HelveticaNeue-Bold - 16pt
    - UIFont.titleFont = HelveticaNeue-Bold - 34pt
    - UIFont.subTitleFont = HelveticaNeue-Bold - 16pt

## Notes:
     I’m relatively new to SKNode’s physic bodies so I’m sure there is a better way to do the spinning circle and spring animations using functionality included in that library.  Optionally we could also use less delegation and put everything in the scene.