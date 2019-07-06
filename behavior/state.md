# 상태 패턴

## Boost.MSM

```cpp
struct PhoneStateMachine : boost::msm::state_machine_def<PhoneStateMachine>
{
    bool angry{ false };
    struct OffHook : boost::msm::state<>
    {};
    struct Connecting : boost::msm::state<>
    {
        template<class Event, class FSM>
        void on_entry(Event const & evt, FSM &)
        {
            std::cout << "We are connecting...\n";
        }
    };

    ...
};

...

struct PhoneBeingDestroyed
{
    template<class EVT, class FSM, class SourceTarget, class TargetState>
    void operator () (EVT const &, FSM &, SourceTarget &, TargetState &)
    {
        std::cout << "Phone breaks into a million piecesn";
    }
};

struct CanDestroyPhone
{
    template<class EVT, class FSM, class SourceTarget, class TargetState>
    bool operator () (EVT const &, FSM & fsm, SourceTarget &, TargetState &)
    {
        return fsm.angry;
    }
};

struct transition_table : 
    mpl::vector<
        row<OffHook, CallDialed, Connecting>,
        row<Connecting, CallConnected, Connected>,
        row<OnHold, PhoneThrownIntoWall, PhoneDestoryed, PhoneBeingDestroyed, CanDestoryPhone>
    >
{};

using initial_state = OffHook;

template<class FSM, class Event>
void no_transition(Event const & e, FSM &, int state)
{
    std::cout << "No transition from state " << state_names[state] <<
    " on event " << typeid(e).name() << "\n";
}

boost::msm::back::state_machine<PhoneStateMachine> phone;
```
