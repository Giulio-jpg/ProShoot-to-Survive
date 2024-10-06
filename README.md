# About

This project was done as the final test of A.I.V.'s Unity course in june 2023.

It's a very arcade Third Person Shooter made with Unity in which you must fight off several waves of zombies in order to win.

In addition to the default gun there are power ups and also another weapon that can be acquired by killing zombies.

One particular mechanic we have tried to implement are two bars that are replenished by killing two different types of enemies, melee and ranged. If you kill a melee enemy the corresponding bar will fill up, and the other bar will decrease a bit. When the bars reach a certain threshold, they will give the player a very strong upgrade.

There is a scoring system that decreases over time, and increases based on kills or bar state.

Here a short gameplay video:



https://github.com/user-attachments/assets/bea9c80a-d5c9-4818-aef0-c84f1ec36ab3



The purpose was to combine all the Unity knowledge acquired during the course into a single project, creating something more organic and structured, following a well-defined development process

![photo_2024-07-27_18-22-41](https://github.com/user-attachments/assets/0ec55e6a-e585-46f6-8f15-916cc41ff825)

It was a challenging and fun experience wich I worked on with 4 other students from my programming course.

I worked mainly on the UI (both design and functionalities), bullets logic, power ups, score system and audio. I also did the level design of the map.

## Bullets

Both player and enemy bullets are handled by a manager class with [object pooling design pattern](https://en.wikipedia.org/wiki/Object_pool_pattern)

Since the game is very arcade and the bullets are not too fast, I thought it appropriate to avoid using raycast (since they are expensive in terms of performance) to check bullets collisions, which are instead simply checked through the OnTriggerEnter events of the colliders, like:

``` cs 
protected override void OnTriggerEnter(Collider other)
{
    if (other.CompareTag("Player"))
    {
        PlayerLogic player = other.gameObject.GetComponent<PlayerLogic>();
        player.TakeDamage(DamageDealt);
    }
    WeaponManager.Instance.ReturnEnemyBullet(gameObject);
}
```

Movement of bullets is defined as follows:

``` cs 
protected void StandardMovement()
{
    transform.position += dir * bulletSpeed * Time.deltaTime;
}
```

It is possible, however, to pick up a power up that makes bullets homing for a certain period of time, in which case their movement must change.

Before firing, the closest enemy within the player's range of vision is found and the bullet will be bound to follow him.

``` cs 
private void HomingMovement()
{
    if (target != null)
    {
        if (homingTimer <= 0)
        {
            Vector3 distance = target.position - transform.position;
            Quaternion rotation = Quaternion.LookRotation(distance);
            Quaternion newRotation = Quaternion.Lerp(transform.rotation, rotation, rotSpeed * Time.deltaTime);
            transform.rotation = newRotation;
        }
    }
    transform.position += transform.forward * bulletSpeed * Time.deltaTime;
}
```

 

That homingTimer indicates the time that must elapse before the bullet follow the enemy, a choice made to simulate the initial impulse of the shot 


https://github.com/user-attachments/assets/ee7c29b6-65ec-4570-b7e2-68fabbc7eee5

## Power ups

There are 4 different power ups: one heals half of life, one adds an amount of armor (up to a maximum limit), one makes bullets homing and the fourth is a grenade.

When the grenade explodes it not only does damage to enemies but also pushes them away

``` cs                                  
private void Explode()
{
    Instantiate(explosionFX, transform.position, transform.rotation);

    Collider[] explodingColliders = Physics.OverlapSphere(transform.position, explosionRadius, enemyMask);

    foreach (Collider collider in explodingColliders)
    {
        EnemyLogic enemy = collider.GetComponent<EnemyLogic>();
        enemy.ReceiveDamage(damage);

        EnemyAI enemyAI = collider.GetComponent<EnemyAI>();
        if (enemyAI.IsAlive)
        {
            enemyAI.DisableAgent();
            enemyAI.rb.AddExplosionForce(explosionForce, transform.position, explosionRadius);
        }
        
    }

    Destroy(gameObject);
}
```

However, since the enemies are moved by the navmesh agent I thought of temporarily deactivating the agent so that the explosion would push them

``` cs 
public void DisableAgent()
{
    rb.isKinematic = false;
    enemyAgent.enabled = false;
    enemyAnimator.enabled = false;
    Invoke(nameof(EnableAgent), reactivationTime);
}
```

Video: 


https://github.com/user-attachments/assets/a41ea064-91d4-45c1-85f0-3270f45066ff

> **_Rotating effect:_** I also made a class that takes care of rotating all objects dropped by enemies to make them more visible. In its Update it will take care of rotating all the objects it has registered.




