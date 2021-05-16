[Front page](./index.html)

# Object Interactions

Is a project where the player is controlled from a first-person perspective and player can interact with different objects.

I did this project for Advanced Game Programming -course.

[Link to Object Interactions GitHub](https://github.com/Eetui/ObjectInteractions)


## Goals of the course
My goal was to write modular and cleaner code.

## What did I learn?

I fulfilled my goal and managed to write modular and cleaner code. I learned SOLID-principles on some level and how to implement them into my code. I also learned how to implement events with Scriptable Objects and how to use Scriptable Objects overall. In addition I used interfaces on this project and became more familiar with them. And thousand little things what comes when making a project.

## Code and Gameplay


### Player Controls

First thing I implemented was the player controls. It was done with Unitys old input system and CharacterController.

#### PlayerMovement.cs
```cs
private void Update()
{
    speed = Input.GetKey(KeyCode.LeftShift) ? runningSpeed : walkingSpeed;
    Vector2 movementInput = new Vector2(Input.GetAxisRaw("Horizontal"), Input.GetAxisRaw("Vertical"));

    movementInput.Normalize(); // Normalizing the input so walking diagonally isn't faster.

    if (groundCheck.IsGrounded() && velocity.y < 0f) velocity.y = -1f;

    velocity.y += gravity * Time.deltaTime; // Falling speed increases overtime. if check above resets this if grounded.

    Vector3 movement = ((transform.forward * movementInput.y) + (transform.right * movementInput.x)) * speed + (Vector3.up * velocity.y);
    characterController.Move(movement * Time.deltaTime);
}
```
#### PlayerLook.cs

```cs
private void Update()
{
    Vector2 mouseInput = new Vector2(Input.GetAxis("Mouse X"), Input.GetAxis("Mouse Y"));
    
    CalculateCameraPitch(mouseInput.y);

    //yaw
    transform.Rotate(Vector3.up * mouseInput.x * sensitivity);
}

private void CalculateCameraPitch(float mouseInputY)
{
    if (invertedLook) cameraPitch += mouseInputY * sensitivity;
    else cameraPitch -= mouseInputY * sensitivity;

    cameraPitch = Mathf.Clamp(cameraPitch, -90f, 90f);
    
    playerCamera.localEulerAngles = Vector3.right * cameraPitch;
}
```
