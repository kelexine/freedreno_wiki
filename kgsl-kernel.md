### General information

From libdrm_freedreno's README:

Current msm kernel driver is a bit strange.  It provides a
DRM interface for GEM, which is basically sufficient to have DRI2
working.  But it does not provide KMS.  And interface to 2d and 3d
cores is via different other devices (/dev/kgsl-*).

### Drawing contexts

Drawing contexts are created via IOCTL_KGSL_DRAWCTXT_CREATE, it requires a single argument -
set of context flags, these are:

    #define KGSL_CONTEXT_SAVE_GMEM 0x00000001
    #define KGSL_CONTEXT_NO_GMEM_ALLOC 0x00000002
    #define KGSL_CONTEXT_SUBMIT_IB_LIST 0x00000004
    #define KGSL_CONTEXT_CTX_SWITCH 0x00000008
    #define KGSL_CONTEXT_PREAMBLE 0x00000010
    #define KGSL_CONTEXT_TRASH_STATE 0x00000020
    #define KGSL_CONTEXT_PER_CONTEXT_TS 0x00000040
    #define KGSL_CONTEXT_USER_GENERATED_TS 0x00000080
    #define KGSL_CONTEXT_NO_FAULT_TOLERANCE 0x00000200
    #define KGSL_CONTEXT_TYPE_MASK 0x01F00000
    #define KGSL_CONTEXT_TYPE_SHIFT 20
    #define KGSL_CONTEXT_TYPE_ANY 0
    #define KGSL_CONTEXT_TYPE_GL 1
    #define KGSL_CONTEXT_TYPE_CL 2
    #define KGSL_CONTEXT_TYPE_C2D 3
    #define KGSL_CONTEXT_TYPE_RS 4
    #define KGSL_CONTEXT_TYPE_UNKNOWN 0x1E

Freedreno creates a single drawing context for each pipe, i.e:

    struct fd_pipe * kgsl_pipe_new(struct fd_device *dev, enum fd_pipe_id id)
    {
        struct kgsl_drawctxt_create req = {
            .flags = 0x2000, /* ??? */
        };
        ...
        ret = ioctl(fd, IOCTL_KGSL_DRAWCTXT_CREATE, &req);
    }

it doesn't set any flags currently, but you can change that if you have problems with
your particular kgsl kernel driver (see below)

### IB submission

IB submission is done via IOCTL_KGSL_RINGBUFFER_ISSUEIBCMDS, user defines an IB chain using
struct kgsl_ibdesc like this:

    struct kgsl_ibdesc ibdesc[] = {
        {
            .gpuaddr = kgsl_ring->bo->gpuaddr + offset,
            .hostptr = last_start,
            .sizedwords = ring->cur - last_start,
        },
        {
            ...
        },
    };
    struct kgsl_ringbuffer_issueibcmds req = {
         .drawctxt_id = kgsl_pipe->drawctxt_id,
         .ibdesc_addr = (unsigned long)&ibdesc,
         .numibs = sizeof(ibdesc)/sizeof(ibdesc[0]),
         .flags = KGSL_CONTEXT_SUBMIT_IB_LIST,
    };

Depending on drawing context flags kgsl will handle IBs differently:
 * By default IBs are executed as is, i.e. one after another
 * If KGSL_CONTEXT_PREAMBLE was specified then IB 0 is handled differently, it doesn't get
   executed right away, instead it serves as a state-restoration IB, when a context changes
   inside kgsl kernel driver it'll first execute IB 0 to setup the state and then move on
   to IB 1, 2, so on.

The default way has its own implications, it builds gpu state shadow and uses it to save
context information, some kernel drivers have it broken and qcom blob always uses the preamble
approach. But since currently freedreno always uses single IB and provides enough information
to setup the gpu state you can easily change command submission to use preamble by providing
IB 0 with a single CP_NOP instruction, i.e. in kgsl_ringbuffer_new:

    kgsl_ring->tmp_bo = kgsl_rb_bo_new(to_kgsl_pipe(pipe), 0x10000);

    ptr = kgsl_ring->tmp_bo->hostptr;

    /*
     * CP_NOP instruction with payload of 1 dword.
     */

    *ptr = 0xc0001000;
    *(ptr + 1) = 00000000;

then in kgsl_ringbuffer_flush:

    struct kgsl_ibdesc ibdesc[2] = {
        {
            .gpuaddr = kgsl_ring->tmp_bo->gpuaddr,
            .hostptr = NULL,
            .sizedwords = 2,
        },
        {
            .gpuaddr = kgsl_ring->bo->gpuaddr + offset,
            .hostptr = last_start,
            .sizedwords = ring->cur - last_start,
        }
    };
    struct kgsl_ringbuffer_issueibcmds req = {
        .drawctxt_id = kgsl_pipe->drawctxt_id,
        .ibdesc_addr = (unsigned long)&ibdesc,
        .numibs = 2,
        .flags = KGSL_CONTEXT_SUBMIT_IB_LIST,
    };

And of course, you'll need to set KGSL_CONTEXT_PREAMBLE flag in kgsl_pipe_new