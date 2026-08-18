## Contributing

Hi there! We're thrilled that you'd like to contribute to this project. Your help is essential for keeping it great.

Contributions to this project are [released](https://help.github.com/articles/github-terms-of-service/#6-contributions-under-repository-license) to the public under the project's open source license.

Please note that this project is released with a Contributor Code of Conduct. By participating in this project you agree to abide by its terms.

## Submitting a pull request

0. Fork and clone the repository
0. Configure and install the dependencies: `script/bootstrap`
0. Make sure the tests pass on your machine: `script/cibuild`
0. Create a new branch: `git checkout -b my-branch-name`
0. Make your change, add tests, and make sure the tests still pass
0. Push to your fork and submit a pull request
0. Pat your self on the back and wait for your pull request to be reviewed and merged.

Here are a few things you can do that will increase the likelihood of your pull request being accepted:

- Follow standards for style and code quality.
- Write tests.
- Keep your change as focused as possible. If there are multiple changes you would like to make that are not dependent upon each other, consider submitting them as separate pull requests.
- Write a [good commit message](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html).

## Resources

- [How to Contribute to Open Source](https://opensource.guide/how-to-contribute/)
- [Using Pull Requests](https://help.github.com/articles/about-pull-requests/)
- [GitHub Help](https://help.github.com)
using UnityEngine;

public class FootballPlayerController : MonoBehaviour
{
    [Header("Player Movement")]
    public float moveSpeed = 5f;
    public float rotationSpeed = 10f;

    [Header("Football")]
    public Rigidbody football;
    public Transform ballPosition;

    [Header("Kick Power")]
    public float passPower = 7f;
    public float shootPower = 14f;

    private Rigidbody playerRigidbody;
    private Vector3 movement;

    void Start()
    {
        playerRigidbody = GetComponent<Rigidbody>();

        if (playerRigidbody != null)
        {
            playerRigidbody.constraints =
                RigidbodyConstraints.FreezeRotationX |
                RigidbodyConstraints.FreezeRotationZ;
        }
    }

    void Update()
    {
        // Keyboard controls for testing in Unity
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        movement = new Vector3(horizontal, 0f, vertical);

        if (movement.magnitude > 0.1f)
        {
            movement.Normalize();

            Quaternion targetRotation =
                Quaternion.LookRotation(movement);

            transform.rotation = Quaternion.Slerp(
                transform.rotation,
                targetRotation,
                rotationSpeed * Time.deltaTime
            );
        }
    }

    void FixedUpdate()
    {
        if (playerRigidbody == null)
            return;

        Vector3 newPosition =
            playerRigidbody.position +
            movement * moveSpeed * Time.fixedDeltaTime;

        playerRigidbody.MovePosition(newPosition);
    }

    // PASS BUTTON
    public void PassBall()
    {
        if (football == null)
            return;

        KickBall(passPower);
    }

    // SHOOT BUTTON
    public void ShootBall()
    {
        if (football == null)
            return;

        KickBall(shootPower);
    }

    void KickBall(float power)
    {
        // Check that the ball is close enough
        float distance = Vector3.Distance(
            transform.position,
            football.transform.position
        );

        if (distance > 3f)
        {
            Debug.Log("Ball is too far away!");
            return;
        }

        // Move ball slightly in front of player
        if (ballPosition != null)
        {
            football.transform.position =
                ballPosition.position;
        }

        // Remove previous movement
        football.linearVelocity = Vector3.zero;
        football.angularVelocity = Vector3.zero;

        // Kick the ball forward
        Vector3 kickDirection =
            transform.forward + Vector3.up * 0.08f;

        football.AddForce(
            kickDirection.normalized * power,
            ForceMode.Impulse
        );
    }
}
