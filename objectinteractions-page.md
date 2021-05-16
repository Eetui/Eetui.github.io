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

### Interactables

All interactable objects inherit IInteractable interface. IInteractable methdos are called from PlayerInteract class, which is attached to the player.

#### IInteractable.cs

```cs
public interface IInteractable
{
    string GetInteractionText();
    void Interact();
}
```

Example class that inherits IInteractable

#### InteractableDoor.cs

```cs
public class InteractableDoor : Door, IInteractable
{
    public string GetInteractionText() => isDoorOpen ? "Open Door" : "Close Door";

    public void Interact() => UseDoor();
}
```

![Interactions](https://j.gifs.com/K8X6oY.gif)

All this is possible because of the PlayerInteract class that I mentioned earlier.

#### PlayerInteract.cs

```cs

public IInteractable interactable = null; // interactable variable

... // skipping some code

private void Update()
{
    interactable = null;

    Physics.Raycast(camTransform.position, camTransform.TransformDirection(Vector3.forward), out var hit, interactRange, interactableLayerMask);
    var hitTransform = hit.transform;

    if (hitTransform != null)
    {
        if (hitTransform.TryGetComponent(out interactable) && groundCheck.IsGrounded())
        {
            if (Input.GetKeyDown(interactionKey))
            {
                interactable.Interact();
                onInteractionChanged?.Invoke();
            }
        }
    }
... // code goes on

```

PlayerInteract is shooting a ray and if the ray hits some object, it will look if the object inherits IInteractable. If the object inherits from IInteractable it will store the information to variable called interactable. Then if player presses the interaction key, it will call the objects IInteractable interact method.

For example if the player looks at the InteractableDoor, the PlayerInteract script will look if the InteractableDoor has inherited the IInteraction interface and since it has, the PlayerInteract script will continue and wait for the player input. Then the player presses the interaction key and the PlayerInteract script will call the InteractableDoor's Interact method, which will call the UseDoor method and the door opens.

### Interactable UI

#### UIInteract.cs

```cs
public class UIInteract : MonoBehaviour
{
    [SerializeField] private PlayerInteract player;

    [Header("Interaction")]
    [SerializeField] private GameObject interactPanel;
    [SerializeField] private Text interactKeyText;
    [SerializeField] private Text interactionText;
    private string interactionKey;

    private void Start() => interactionKey = player.InteractionKey.ToString();

    public void UpdateInteractionUI()
    {
        if (player.interactable != null)
        {
            interactionText.text = player.interactable.GetInteractionText();
            interactKeyText.text = interactionKey;
            interactPanel.SetActive(true);
        }
        else
        {
            interactPanel.SetActive(false);
        }
    }
}
```
