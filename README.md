# hatchbed_localization

ROS 2 helper nodes for TF tree management and localization support.

## Nodes

### imu_to_twist

Converts a `sensor_msgs/Imu` message to `geometry_msgs/TwistWithCovarianceStamped` for use
as a velocity input to state estimators such as `robot_localization`.

In direct mode the angular velocity and its diagonal covariance are copied from the IMU
message.  Linear velocity is always zero (IMUs do not provide it).

In differential mode the angular velocity is derived by finite-differencing consecutive
orientation quaternions.  The orientation field must be populated
(`orientation_covariance[0] >= 0`).

**Subscriptions**

| Topic | Type | Description |
|---|---|---|
| *(imu_topic)* | `sensor_msgs/Imu` | Input IMU messages |

**Publications**

| Topic | Type | Description |
|---|---|---|
| `twist` | `geometry_msgs/TwistWithCovarianceStamped` | Angular velocity derived from IMU |

**Parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `imu_topic` | string | *required* | IMU topic to subscribe to |
| `differential` | bool | `false` | Derive angular velocity from orientation differential rather than reading angular_velocity directly |

---

### odom_to_twist

Extracts the twist from a `nav_msgs/Odometry` message and republishes it as
`geometry_msgs/TwistWithCovarianceStamped`.  Useful when a state estimator requires a
standalone twist topic rather than a full odometry message.

In direct mode the twist field is copied verbatim.  In differential mode the body-frame
twist is derived by finite-differencing consecutive poses, with covariance propagated as
`C_twist = 2 * C_pose / dt^2`.

**Subscriptions**

| Topic | Type | Description |
|---|---|---|
| *(odom_topic)* | `nav_msgs/Odometry` | Input odometry messages |

**Publications**

| Topic | Type | Description |
|---|---|---|
| `twist` | `geometry_msgs/TwistWithCovarianceStamped` | Twist extracted from odometry |

**Parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `odom_topic` | string | *required* | Odometry topic to subscribe to |
| `frame_id` | string | `""` | Override the output header frame_id; uses the odometry frame if empty |
| `differential` | bool | `false` | Derive twist by differencing consecutive poses rather than reading the twist field directly |

---

### odom_tf_broadcast

Derives and broadcasts a `parent_frame -> child_frame` TF transform from an odometry
message.  Intended for cases where a localization source publishes an odometry topic but
its TF output is suppressed or absent, or where the desired TF relationship differs from
the one directly expressed in the odometry frames.

`child_frame` must match either `header.frame_id` or `child_frame_id` of the incoming
odometry message.  The other frame is looked up in the main TF tree to compute the
transform.  The `parent_frame == known_frame` identity case is handled without a TF
lookup, so `parent_frame` may appear directly in the odometry message.

The only hard requirement is that `child_frame` must not already have a parent in the
main TF tree.

**Subscriptions**

| Topic | Type | Description |
|---|---|---|
| *(odom_topic)* | `nav_msgs/Odometry` | Input odometry messages |

**Publications**

| Topic | Type | Description |
|---|---|---|
| `/tf` | `tf2_msgs/TFMessage` | `parent_frame -> child_frame` transform |

**Parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `odom_topic` | string | *required* | Odometry topic to subscribe to |
| `child_frame` | string | *required* | Frame to broadcast as a child of `parent_frame`; must appear in the odometry message |
| `parent_frame` | string | *required* | Frame that will become the parent of `child_frame` in the main TF tree |
| `timestamp_offset` | double | `0.0` | Seconds added to the message timestamp before broadcasting; a positive value future-dates the transform to keep it valid between low-frequency updates |

`timestamp_offset` is dynamically reconfigurable at runtime.

---

### tf_reroot

Bridges two otherwise disconnected TF trees to broadcast `parent_frame -> child_frame`
on the main `/tf` topic.

The source TF topic carries transforms that are intentionally kept off the main tree
(typically by remapping `/tf` to a private topic in the publisher's launch configuration).
The node automatically detects a pivot frame — any frame reachable from `child_frame` in
one buffer and from `parent_frame` in the other — then computes:

```
T_parent_child = T_parent_pivot * inv(T_child_pivot)
```

Either `child_frame` or `parent_frame` may originate from the source topic; the node
searches both buffers for each endpoint when looking for a pivot.  If multiple pivot
frames are found a warning is logged and the first one is used.  The only hard
requirement is that `child_frame` must not already have a parent in the main TF tree.

**Subscriptions**

| Topic | Type | Description |
|---|---|---|
| *(source_tf_topic)* | `tf2_msgs/TFMessage` | Source TF transforms, kept separate from the main tree |
| `/tf` | `tf2_msgs/TFMessage` | Main TF tree (via TransformListener) |
| `/tf_static` | `tf2_msgs/TFMessage` | Static transforms (via TransformListener) |

**Publications**

| Topic | Type | Description |
|---|---|---|
| `/tf` | `tf2_msgs/TFMessage` | `parent_frame -> child_frame` bridging transform |

**Parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `child_frame` | string | *required* | Frame to broadcast as a child of `parent_frame` |
| `parent_frame` | string | *required* | Frame that will become the parent of `child_frame` in the main TF tree |
| `source_tf_topic` | string | *required* | TF topic carrying the source transforms |
