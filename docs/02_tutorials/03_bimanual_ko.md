# 양팔(Bimanual) 워크플로

**Overview**
양팔(dual-arm)에서는 `bi_so_follower` / `bi_so_leader`를 사용합니다. 흐름은 한팔과 동일하지만, 좌/우 팔 포트와 좌/우 카메라 구성을 각각 전달해야 합니다.

**Prereqs**
아래 값을 실제 장치 경로로 바꿔서 사용하세요.
- `LEFT_FOLLOWER_PORT` 예: `/dev/follower_arm_left`
- `RIGHT_FOLLOWER_PORT` 예: `/dev/follower_arm_right`
- `LEFT_LEADER_PORT` 예: `/dev/leader_arm_left`
- `RIGHT_LEADER_PORT` 예: `/dev/leader_arm_right`
- `LEFT_CAM_WRIST` 예: `1` 또는 `/dev/left_wrist_cam`
- `RIGHT_CAM_WRIST` 예: `2` 또는 `/dev/right_wrist_cam`
- `TOP_CAM` 예: `3` 또는 `/dev/top_cam`
- `FRONT_CAM` 예: `4` 또는 `/dev/front_cam`

**Teleoperate (No Record)**
데이터 저장 없이 로봇 동작을 테스트하거나 카메라 화면을 확인할 때 사용합니다.

```bash
lerobot-teleoperate \
  --robot.type=bi_so_follower \
  --robot.left_arm_config.port=LEFT_FOLLOWER_PORT \
  --robot.right_arm_config.port=RIGHT_FOLLOWER_PORT \
  --robot.id=bimanual_follower \
  --robot.left_arm_config.cameras='{"wrist":{"type":"opencv","index_or_path":LEFT_CAM_WRIST,"width":640,"height":480,"fps":30},"top":{"type":"opencv","index_or_path":TOP_CAM,"width":640,"height":480,"fps":30}}' \
  --robot.right_arm_config.cameras='{"wrist":{"type":"opencv","index_or_path":RIGHT_CAM_WRIST,"width":640,"height":480,"fps":30},"front":{"type":"opencv","index_or_path":FRONT_CAM,"width":640,"height":480,"fps":30}}' \
  --teleop.type=bi_so_leader \
  --teleop.left_arm_config.port=LEFT_LEADER_PORT \
  --teleop.right_arm_config.port=RIGHT_LEADER_PORT \
  --teleop.id=bimanual_leader \
  --display_data=true
```

**Record (Teleop)**
양팔 데이터 기록은 teleop이 사실상 필수입니다. 아래 명령으로 데이터셋을 생성합니다.

```bash
lerobot-record \
  --robot.type=bi_so_follower \
  --robot.left_arm_config.port=LEFT_FOLLOWER_PORT \
  --robot.right_arm_config.port=RIGHT_FOLLOWER_PORT \
  --robot.id=bimanual_follower \
  --robot.left_arm_config.cameras='{"wrist":{"type":"opencv","index_or_path":LEFT_CAM_WRIST,"width":640,"height":480,"fps":30},"top":{"type":"opencv","index_or_path":TOP_CAM,"width":640,"height":480,"fps":30}}' \
  --robot.right_arm_config.cameras='{"wrist":{"type":"opencv","index_or_path":RIGHT_CAM_WRIST,"width":640,"height":480,"fps":30},"front":{"type":"opencv","index_or_path":FRONT_CAM,"width":640,"height":480,"fps":30}}' \
  --teleop.type=bi_so_leader \
  --teleop.left_arm_config.port=LEFT_LEADER_PORT \
  --teleop.right_arm_config.port=RIGHT_LEADER_PORT \
  --teleop.id=bimanual_leader \
  --display_data=true \
  --dataset.repo_id=jinhyuk2me/bimanual-so-handover-cube \
  --dataset.num_episodes=25 \
  --dataset.single_task="Grab and handover the red cube to the other arm"
```

**Train**

```bash
lerobot-train \
  --dataset.repo_id=jinhyuk2me/bimanual-so-handover-cube \
  --dataset.video_backend=pyav \
  --policy.type=act \
  --output_dir=outputs/train/act_bimanual \
  --job_name=act_bimanual \
  --policy.device=cuda \
  --policy.repo_id=jinhyuk2me/act_bimanual
```

**Eval (Policy Inference)**
학습된 정책으로 양팔 평가를 기록합니다.

```bash
lerobot-record \
  --robot.type=bi_so_follower \
  --robot.left_arm_config.port=LEFT_FOLLOWER_PORT \
  --robot.right_arm_config.port=RIGHT_FOLLOWER_PORT \
  --robot.id=bimanual_follower \
  --robot.left_arm_config.cameras='{"wrist":{"type":"opencv","index_or_path":LEFT_CAM_WRIST,"width":640,"height":480,"fps":30},"top":{"type":"opencv","index_or_path":TOP_CAM,"width":640,"height":480,"fps":30}}' \
  --robot.right_arm_config.cameras='{"wrist":{"type":"opencv","index_or_path":RIGHT_CAM_WRIST,"width":640,"height":480,"fps":30},"front":{"type":"opencv","index_or_path":FRONT_CAM,"width":640,"height":480,"fps":30}}' \
  --policy.path=/home/jinhyuk2me/dev_ws/mt_lerobot/outputs/train/act_bimanual/checkpoints/last/pretrained_model \
  --teleop.type=bi_so_leader \
  --teleop.left_arm_config.port=LEFT_LEADER_PORT \
  --teleop.right_arm_config.port=RIGHT_LEADER_PORT \
  --teleop.id=bimanual_leader \
  --display_data=true \
  --dataset.repo_id=jinhyuk2me/eval_act_bimanual \
  --dataset.num_episodes=10 \
  --dataset.single_task="Grab and handover the red cube to the other arm"
```

**Notes**
- 리셋 구간(`--dataset.reset_time_s`)에 teleop이 없으면 `No policy or teleoperator provided...` 경고가 반복될 수 있습니다.
- 빠른 테스트는 `--dataset.num_episodes=1`로 시작하는 것을 권장합니다.
