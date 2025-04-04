QEMU ?= qemu-system-x86_64
QEMU-UINTR ?= qemu-system-x86_64-uintr
PORT ?= 2333
PORT_UINTR ?= 2334

qemu_image := ubuntu-20.04-server-cloudimg-amd64.img
qemu_args := \
  -smp 4 -m 4G -accel kvm -nographic \
  -machine q35,kernel_irqchip=split \
  -cpu host,-kvm-asyncpf,-kvm-pv-eoi,-kvm-pv-ipi,-kvm-pv-sched-yield,-kvm-pv-unhalt,-kvm-steal-time,-kvmclock \
  -drive file=$(qemu_image) \
  -net user,id=net,hostfwd=tcp::$(PORT)-:22 -net nic,model=e1000e \
  -serial mon:stdio

qemu_image_ubuntu22_download := ubuntu-22.04-server-cloudimg-amd64.img
qemu_image_ubuntu22_bak := ubuntu22_bak.img
qemu_image_uintr := ubuntu22_uintr.img

qemu_args_uintr := \
  -smp 4 -m 4G -nographic \
  -machine q35,kernel_irqchip=split \
  -cpu qemu64 \
  -drive file=$(qemu_image_uintr) \
  -net user,id=net,hostfwd=tcp::$(PORT_UINTR)-:22 -net nic,model=e1000e \
  -serial mon:stdio \

qemu_args_uintr_accel := \
  -smp 4 -m 8G -accel kvm -nographic \
  -machine q35,kernel_irqchip=split \
  -cpu host,-kvm-asyncpf,-kvm-pv-eoi,-kvm-pv-ipi,-kvm-pv-sched-yield,-kvm-pv-unhalt,-kvm-steal-time,-kvmclock \
  -drive file=$(qemu_image_uintr) \
  -net user,id=net,hostfwd=tcp::$(PORT_UINTR)-:22 -net nic,model=e1000e \
  -serial mon:stdio \



$(qemu_image):
	wget https://cloud-images.ubuntu.com/releases/focal/release/$(qemu_image)

.ONESHELL:
image: $(qemu_image)`
	cat >user-data <<EOF
	#cloud-config
	password: guest
	chpasswd: { expire: False }
	ssh_pwauth: True
	EOF
	cloud-localds user-data.img user-data
	qemu-img resize $(qemu_image) +64G
	$(QEMU) $(qemu_args) -drive file=user-data.img,format=raw


ubuntu22:
	wget https://cloud-images.ubuntu.com/releases/jammy/release/$(qemu_image_ubuntu22_download) -O $(qemu_image_ubuntu22)

user-data-uintr:
	cat >user-data <<EOF
	#cloud-config
	chpasswd:  
		list: |
			root:guest
			ubuntu:guest
		expire: False
	ssh_pwauth: True
	EOF
	cloud-localds user-data-uintr.img user-data-uintr

uintr_image: ubuntu22 user-data-uintr
	qemu-img create -f qcow2 -b $(qemu_image_ubuntu22_bak) -F qcow2 ubuntu22_uintr.img 40G

uintr:
	$(QEMU-UINTR) $(qemu_args_uintr)

uintr_accel:
	$(QEMU-UINTR) $(qemu_args_uintr_accel)

uintr_firstrun: 
	$(QEMU-UINTR) $(qemu_args_uintr_accel) -drive file=user-data-uintr.img,format=raw

qemu: $(qemu_image)
	$(QEMU) $(qemu_args)

ssh:
	ssh -p $(PORT) ubuntu@localhost

ssh_uintr:
	ssh -p $(PORT_UINTR) ubuntu@localhost